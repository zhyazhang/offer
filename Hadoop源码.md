## 1.Hadoop集群和Yarn

虽然Hadoop也可以在单机上运行,但是这个平台的典型运行场景是在多机的集群(Cluster)上。把运行着Hadoop平台的集群,就Hadoop平台的边界所及,称为“Hadoop集群”。其中的每台机器都成为集群的一个“节点(node)”,节点之间连成一个局域网。这个局域网一般是交换网,而不是路由网。这就是说,集群中只有交换机(switch),一般是二层交换机,也可能是三层交换机,但是没有普通的路由器,因为那些路由器引入的延迟太大。不过这也不绝对,有时候可能确实需要将一个集群分处在不同网段中,而通过路由器相连,但是这并不影响 Hadoop的运行(除性能降低之外)。就 Hadoop而言,路由器与交换机在逻辑上是一样的。

集群内的节点之间可以通过IP地址通信,也可以通过节点的域名即URL通信,这就需要有DNS的帮助。在网络可以通达的某处存在着DNS服务,因而可以根据对方的URL查到其IP地址。集群内这些节点都应有个域名,并且登记在DNS中。实际上节点间还可以通过节点名通信,但是那跟URL本质上是一样,最后总是转换成IP地址。不管是静态设定还是通过DHCP动态分配,每个节点都必须有个IP地址。

Hadoop的有些操作需要知道具体节点的IP地址、域名之类的信息,需要直接与DNS交互,为此Hadoop定义了一个名为DNS的类,用来提供帮助

```java
//org.apache.hadoop.net

public class DNS {

  private static final Logger LOG = LoggerFactory.getLogger(DNS.class);
    
    /**
    *缓存hostname，初始化为null
    */
  private static final String cachedHostname = resolveLocalHostname();
  private static final String cachedHostAddress = resolveLocalHostIPAddress();
  private static final String LOCALHOST = "localhost";

  /**
   *通过提供的名称服务器返回与指定 IP 地址关联的主机名
   */
  public static String reverseDns(InetAddress hostIp, @Nullable String ns)
    throws NamingException {}

  /**
   * @return NetworkInterface for the given subinterface name (eg eth0:0)
   *    or null if no interface with the given name can be found  
   */
  private static NetworkInterface getSubinterface(String strInterface)
      throws SocketException {}

  /**
   * @param nif network interface to get addresses for
   * @return set containing addresses for each subinterface of nif,
   *    see below for the rationale for using an ordered set
   */
  private static LinkedHashSet<InetAddress> getSubinterfaceInetAddrs(
      NetworkInterface nif) {}

  /**
   * Like {@link DNS#getIPs(String, boolean), but returns all
   * IPs associated with the given interface and its subinterfaces.
   */
  public static String[] getIPs(String strInterface)
      throws UnknownHostException {
    return getIPs(strInterface, true);
  }

  /**
   * 以文本形式返回与提供的接口关联的所有 IP（如果存在）
   */
  public static String[] getIPs(String strInterface,
      boolean returnSubinterfaces) throws UnknownHostException {}


  /**
   *如果strInterface为“default”，则返回与提供的网络接口或本地主机 IP 关联的第一个可用 IP 地址
   */
  public static String getDefaultIP(String strInterface)
    throws UnknownHostException {
    String[] ips = getIPs(strInterface);
    return ips[0];
  }

  /**
   * Returns all the host names associated by the provided nameserver with the
   * address bound to the specified network interface
   *返回由提供的名称服务器关联的所有主机名，地址绑定到指定的网络接口
   */
  public static String[] getHosts(String strInterface,
                                  @Nullable String nameserver,
                                  boolean tryfallbackResolution)
      throws UnknownHostException {}


  /**
   * Determine the local hostname; retrieving it from cache if it is known
   * If we cannot determine our host name, return "localhost"
   * 确定本地主机名；如果已知，则从缓存中检索它，如果我们无法确定主机名，则返回“localhost“
   */
  private static String resolveLocalHostname() {}


  /**
   * Get the IPAddress of the local host as a string.
   * This will be a loop back value if the local host address cannot be
   * determined.
   * If the loopback address of "localhost" does not resolve, then the system's
   * network is in such a state that nothing is going to work. A message is
   * logged at the error level and a null pointer returned, a pointer
   * which will trigger failures later on the application
   *以字符串形式获取本地主机的 IPAddress。如果无法确定本地主机地址，返回本地环回地址127.0.0.1。
   *如果“localhost”的环回地址没有解析，那么系统的网络无法工作的状态。
   *在错误级别记录一条消息并返回一个空指针，该指针将在稍后触发应用程序故障
   */
  private static String resolveLocalHostIPAddress() {}

  /**
   * Returns all the host names associated by the default nameserver with the
   * address bound to the specified network interface
   * 返回默认名称服务器关联的指定网络接口地址的所有主机名
   * 
   */
  public static String[] getHosts(String strInterface)
    throws UnknownHostException {}

  /**
   * Returns the default (first) host name associated by the provided
   * nameserver with the address bound to the specified network interface
   * 返回由提供的命名服务器关联到指定网络接口的地址的默认(第一个)主机名
   */
  public static String getDefaultHost(@Nullable String strInterface,
                                      @Nullable String nameserver,
                                      boolean tryfallbackResolution)
    throws UnknownHostException {}

  /**
   * Returns the default (first) host name associated by the default
   * nameserver with the address bound to the specified network interface
   */
  public static String getDefaultHost(@Nullable String strInterface)
    throws UnknownHostException {}

  /**
   * Returns the default (first) host name associated by the provided
   * nameserver with the address bound to the specified network interface.
   */
  public static String getDefaultHost(@Nullable String strInterface,
                                      @Nullable String nameserver)
      throws UnknownHostException {}

  /**
   * Returns all the IPs associated with the provided interface, if any, as
   * a list of InetAddress objects.
   *
   */
  public static List<InetAddress> getIPsAsInetAddressList(String strInterface,
      boolean returnSubinterfaces) throws UnknownHostException {}
```

现在的交换机都是可网管的,有自己的IP地址,所以 Hadoop把交换机也看成网络中的节点。而机架也可能是可网管的,或者就让交换机同时代表着机架,因为机架上往往集成着交换机。这样,如果把交换机和机架也看成节点,那么整个局域网或者整个集群的拓扑就是个树状的结构。Hadoop集群中能实际参与数据处理的节点都是这棵树上的叶节点,树的根节点就是整个局域网的入口,通常这是一台路由器。其余的节点则都是“中间节点”或“内部节点”,那都是交换机或机架。

```java
//org.apache.hadoop.net
public class NodeBase implements Node {
  /*路径分隔符*/
  public final static char PATH_SEPARATOR = '/';
  /** Path separator as a string {@value} */
  public final static String PATH_SEPARATOR_STR = "/";
  /** 根的字符串表示 */
  public final static String ROOT = "";
  
  protected String name; //host:port#
  protected String location; //此节点位置的字符串表示
  protected int level; //节点在树的哪一层
  protected Node parent; //父节点
  
  /** Default constructor */
  public NodeBase() {
  }
  
  /** Construct a node from its path
   * @param path 
   *   a concatenation of this node's location, the path seperator, and its name 
   */
  public NodeBase(String path) {}
  
  /** Construct a node from its name and its location
   * @param name this node's name (can be null, must not contain {@link #PATH_SEPARATOR})
   * @param location this node's location 
   */
  public NodeBase(String name, String location) {}
  
  /** Construct a node from its name and its location
   * @param name this node's name (can be null, must not contain {@link #PATH_SEPARATOR})
   * @param location this node's location 
   * @param parent this node's parent node
   * @param level this node's level in the tree
   */
  public NodeBase(String name, String location, Node parent, int level) {}

  /**
   * set this node's name and location
   * @param name the (nullable) name -which cannot contain the {@link #PATH_SEPARATOR}
   * @param location the location
   */
  private void set(String name, String location) {}
  
  /** @return this node's name */
  @Override
  public String getName() {}
  
  /** @return this node's network location */
  @Override
  public String getNetworkLocation() {}
  
  /** Set this node's network location
   * @param location the location
   */
  @Override
  public void setNetworkLocation(String location) {}
  
  /**
   * Get the path of a node
   * @param node a non-null node
   * @return the path of a node
   */
  public static String getPath(Node node) {}

  /**
   * Get the path components of a node.
   * @param node a non-null node
   * @return the path of a node
   */
  public static String[] getPathComponents(Node node) {}

  /** Normalize a path by stripping off any trailing {@link #PATH_SEPARATOR}
  *规范化路径
   */
  public static String normalize(String path) {}
  
```



**拓扑结构，该类表示具有树形层次网络拓扑的计算机集群。例如，一个集群可能由许多装满计算机机架的数据中心组成。在网络拓扑中，叶子代表数据节点（计算机），内部节点代表管理进出数据中心或机架的流量的交换机/路由器**

```java
//org.apache.hadoop.net

public class NetworkTopology {
    
}
```



