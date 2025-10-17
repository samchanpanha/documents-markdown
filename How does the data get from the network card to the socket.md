# How does the data get from the network card to the socket?

**Original** **Cloud Code** [Cloud Code](javascript:void(0);)

 *October 10, 2025, 16:47*

## I. Foreword

**We all know that the data from the network to reach our application program is to first pass the network card, and then go to the iso7 layer protocol to reach our application program, then the specific process is how? This article discusses this.**

This article is based on " **Deep Understanding of Linux Networking** " notes , the book is based on Linux source code version 3.10

## II. Overall flow chart

### Epol kernel preparation phase

1. **Creation phase (epol _ create)**

* **The kernel creates an eventpoll object for the current process, which contains:**
* *** * red-black tree (RBR) * *: Used to manage all monitored file descriptors (i.e. registered sockets).**
* *** * Ready queue (rdllist) * *: Nodes that hold triggered events and wait for user-state processing.**
* *** * Wait queue (wq) * *: Holds the blocked waiting process when no event is ready**

2. **Registration phase (epol _ ctl)**

   **When the user calls epol _ ctl to register the socket:**

* **The kernel creates a red-black tree node epitem for the socket and inserts it into the red-black tree of the eventpoll.**
* **Also mount a wait queue entry (wait _ queue _ entry _ t) in the waiting queue (sk _ wq) of the socket, whose callback function is set to ep _ poll _ callback.**
* **When new data arrives from the socket, ep _ poll _ callback is triggered to add the corresponding epitem to the rdllist ready queue for eventpoll.**

3. **The wait phase （ epoll _ wait ） **When the user calls epoll _ wait :

* **The kernel first checks for readable events in the ready queue of the eventpoll; If so, return immediately.**
* **If no event is ready, the kernel constructs a wait queue item for the current thread, adds it to the wait queue for eventpoll (eventpoll.wq), and then puts the thread to sleep.**
* **When ep _ poll _ callback is triggered and events are added to the ready queue, the associated waiting thread is awakened.**

**At this point, these queues are distributed as shown below**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/6icqdf4vmsoSho5fAIibicE4mviavHu9Ip8vGs5ko3LaIxeO3n114z4iaYbbxATmxzU0JRZnXPAteviaTysUK0jduiaew/640?wx_fmt=png&from=appmsg)

**The current process has a server socket (underlying sock) and an epoll kernel object (eventpoll)**

**The socket has a waiting queue, sk_wq, each node of the queue represents a blocked client request, unlike the bio model * private needs to point to the current thread (because it is called) and is simply empty here (** `<span leaf="" class="js_darkmode__text__81">*private</span>`Points to null), while func points to the callback function (default _ wake _ function) reached by the data, and the node has a base pointer to the red-black tree node object epitem

**The epoll kernel object eventpoll has a waiting queue sk _ wq whose queue elements contain the blocking process and the corresponding callback function**

**The state of the server nodes at this time is as follows**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/6icqdf4vmsoSho5fAIibicE4mviavHu9Ip8v07A3KKicgPSRQO1fYcvB2cfvYibvJSdXjxq2XZicQITu3QgxG5g0oryng/640?wx_fmt=png&from=appmsg)

### Receiving queue of data from network card to sock

![](https://mmbiz.qpic.cn/sz_mmbiz_png/6icqdf4vmsoSho5fAIibicE4mviavHu9Ip8vaf2acsc7Ovol14WCibTWbrE1fQ1x8DXUVxGSEXEb8iaAt7fbiakO2AuSw/640?wx_fmt=png&from=appmsg)

**Quick note:**

1. **Data reaches the card from the network**
2. **DMA  Writes data to kernel space buffer**

   **When the network adapter driver is initialized, a set of**  **ring receive queues （Rx Ring Buffer） ** **,**

   **Each queue item corresponds to a pre-mapped kernel page-box (mapped by the driver via dma _ map _ single).**

   **After the network card receives the data, it passes directly through** **DMA（Direct Memory Access） **Write data to these buffers
3. **Network card triggers hardware interrupt (IRQ)**

   **When the ring buffer is filled with data, the network card sends a hard interrupt signal to the CPU**
4. **CPU Responds to Hard Interrupts**

   **An interrupt handling function runs in an interrupt context and usually does not take out packages directly.**

   **It simply performs lightweight operations （such as logging events, masking interrupts）, and then schedules**  **soft interrupts （softirq） ** **, Specifically, it triggers**  **the NET_RX_SOFTIRQ ** **.**
5. **Soft interrupts are executed by kernel threads (ksoftirqd / napi)**

   **When the soft interrupt is triggered, a poll callback function received by the network is executed (driving the registered napi- > poll).**

   **This step is usually**  **the NAPI mechanism ** **（New API）At work, the driver will actively poll the ring buffer of the network card and take out the data batch**
6. **Read from the DMA buffer and build the skb**

   **The poll function retrieves the data frame from the DMA queue and constructs a** **skb （ skb ） **Object, and then pass it to **the protocol stack** .
7. **Handed over to a network subsystem (the Ethernet layer)**

   **The kernel network subsystem first processes the network header, determines the protocol type (such as IPv4 / IPv6 / ARP), and then calls the corresponding upper protocol handler.**
8. **Neighbor subsystem (neigh layer)**

   **For IP packets, the mapping between MAC and IP is resolved through the Neighbor Subsystem, and the ARP / ND cache is maintained to ensure the correct routing and next hop.**
9. **IP Layer Processing (L3)**

* **Resolve IP headers (TTL, checksum, source / destination address)**
* **Conducting Distribution Restructuring (MTU)**
* **Filtering by netfilter (iptables), routing lookup**
* **The data is then passed to the upper TCP or UDP protocol handler.**

10. **TCP Layer Processing (L4)**

* **Parsing TCP headers (seq, ack, flags, etc.)**
* **Find the corresponding socket according to four tuples (source IP, source port, destination IP, destination port)**
* **Verify serial number and swipe window to update congestion control status**
* **Finally , the data is encapsulated in a skb and added to the socket 's** **receive queue （ sk _ receive _ queue ） .**

### Overall flow chart

![](https://mmbiz.qpic.cn/sz_mmbiz_png/6icqdf4vmsoSho5fAIibicE4mviavHu9Ip8vPn3tbicTnMNoFFcuIrMYhpicibwgVk3Lnk37Dk3t137icg4u8WoZ6oGyaw/640?wx_fmt=png&from=appmsg)

11. **When the data is received, the registered user is called** `<span leaf="" class="js_darkmode__text__230">ep_poll_callback</span>`Function to find the epitem based on the extra base pointer on the waiting task queue, which in turn finds the eventpoll object, and then adds the epitem to the ready queue for eppoll.
12. **If there is a waiting process on the epoll instance**

    **Then check whether there are waiting items in the waiting queue on the eventpoll object (which is set when epoll_wait is executed). If there are no, the soft interrupt is done. If there are a waiting item, find the default_wake_function set in the queue, take out the process descriptor, and wake it**
13. **It then resumes execution from the chokepoint of epoll _ wait (), traverses the rdllist, returns the ready event to user space, and clears the linked list.**

## III. NIO's underlying support process for epoll

**Based on jdk17 (jdk17u-jdk-17.0.7-7)**

```
public class NioServer {
    public static void main(String[] args) throws IOException {

        System.setProperty("java.nio.channels.spi.SelectorProvider", "sun.nio.ch.PollSelectorProvider");

        // 1.创建ServerSocketChannel, 创建内核socket(还有等待队列), 返回fd(在ServerSocketChannel中)
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        // bind(ip和端口与socket绑定) + listen(创建全连接半连接队列)
        serverChannel.bind(new InetSocketAddress(8080));
        // 设置为非阻塞
        serverChannel.configureBlocking(false);

        // 2.创建Selector(mac上是KQueueSelectorImpl), linux是EpollSelectorImpl(对应EPollSelectorProvider)
        Selector selector = Selector.open();

        // 3.给Selector注册监听的ACCEPT事件    (这一步不会调用epoll_ctl将socket添加到epoll中)
        serverChannel.register(selector, SelectionKey.OP_ACCEPT, null);

        System.out.println("服务器启动，监听端口 8080...");

        // 4. 循环监听事件
        while (true) {
            // 1.服务端第一次调用时, 会将监听socket的就绪事件, 添加到selectedKeys集合中 并添加到epoll的红黑树节点上; 这里会调用epoll_ctl方法
            // 2.然后阻塞(调用epoll_wait)直到有事件发生; epoll_wait/kqueue_wait
            selector.select();
//            selector.select(1000);
            // selector.selectNow(); 这个方法不会阻塞, timeout传的是0
            Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();

            // 2.处理完当前所有的就绪事件
            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();
                // 必须移除，避免重复处理
                keyIterator.remove();

                // 三次握手成功的事件(服务端事件)
                if (key.isAcceptable()) {
                    // 处理新的客户端连接
                    ServerSocketChannel server = (ServerSocketChannel) key.channel();
                    // 获取全队列连接; 根据服务端fd获取一个客户端连接的socket对象
                    SocketChannel client = server.accept();
                    client.configureBlocking(false);
                    // 监听读事件; 1.注册读事件到Selector, 将channel和selector绑定生成selectedKey 3.添加到selectedKeys集合中
                    client.register(selector, SelectionKey.OP_READ, null);
                    System.out.println("连接建立：" + client.getRemoteAddress());
                }
                else if (key.isReadable()) {
                    // 处理客户端发送的数据
                    SocketChannel client = (SocketChannel) key.channel();
                    ByteBuffer buffer = ByteBuffer.allocate(1024);

                    int bytesRead = client.read(buffer);
                    if (bytesRead == -1) {
                        // 客户端关闭连接
                        System.out.println("连接关闭：" + client.getRemoteAddress());
                        client.close();
                        key.cancel();
                        continue;
                    }

                    buffer.flip();
                    byte[] data = new byte[buffer.remaining()];
                    buffer.get(data);
                    String message = new String(data);
                    System.out.println("收到消息：" + message);

                    // 回复客户端
                    ByteBuffer response = ByteBuffer.wrap(("服务器收到：" + message).getBytes());
                    client.write(response);
//                    ByteBuffer[] bs = new ByteBuffer[1];
//                    client.write(bs);
                }
            }
        }
    }
}
```

**among**

1. **ServerSocketChannel.open()**

* **In the Java layer, a new file descriptor (fd) is created through IOUtil.newFD (),**
* **The underlying socket () system call creates a TCP socket object in the kernel (struct socket + struct sock)**
* **The returned fd is bound to Java's FileDescriptor.**

2. **serverChannel.bind(new InetSocketAddress(8080))**

* **Call the bind () system call to bind fd to the specified IP and port;**
* **Listen () is then called to create a half-connected queue (SYN queue) and a fully connected queue (accept queue) in the kernel for the socket.**

3. **serverChannel.register(selector, SelectionKey.OP_ACCEPT, null)**

* **Java 层会调用 SelectorImpl.register() → EPollSelectorImpl.implRegister(),**
* **This step in the JDK layer only registers the SelectionKey and interested events into the Selector's internal data structure (keySet / registeredKeys).**

4. **selector.select()**

* **AtEPollSelectorImpl** [#doSelect ](javascript:;)() Will iterate over registeredKeys to find new registered or changed fd's.
* **These fd's are added to the eventpoll red-black tree at the native layer via epol _ ctl (ADD) or MOD.**
* **The benefit of doing this is to**  **batch registration and event modification ** **, avoiding the need to switch to the kernel each time a register is made .**
* **SELECT calls epol _ wait (), returning the ready epitem from the rdllist (ready queue).**
* **The Java layer then encapsulates these events into the corresponding SelectionKeyImpl, which is given to the NIO event loop.**

## IV. CONCLUSIONS

**When a frame of data jumps from a network cable to a network card, its fate is closely linked to the kernel of the operating system.**

1. **The physical layer to the kernel**

* **The network card writes data via DMA to a preallocated kernel buffer (ring queue),**
* **And notifies the CPU through a hard interrupt, triggering a soft interrupt (NET _ RX _ SOFTIRQ), which is processed by a kernel thread (such as ksoftirqd) or a NAPI mechanism for batch polling.**
* **The data is encapsulated as a skb （socket buffer） and traversed sequentially through**  **the Ethernet layer → Neighbor layer → IP layer → TCP / UDP layer ** **,**
* **The TCP layer finds the corresponding socket based on the quad, puts the data into the socket receive queue (sk _ receive _ queue), handles sliding windows and congestion control.**

2. **Kernel Event Mechanism (epoll)**

* **The socket's waiting queue (sk _ wq) registers the epol callback (ep _ poll _ callback), associating the epitem with the eventpoll object.**
* **When the data is ready, ep _ poll _ callback puts the epitem into eppoll's ready queue (rdllist),**
* **And check the waiting queue on the epol instance, and if a thread is blocked at epol _ wait (), call the callback function (default _ wake _ function) to wake it up.**

3. **Mapping and Encapsulation of Java NIO**

* **At the Java layer, ServerSocketChannel.open () creates the fd and corresponds to the kernel socket; Bind () and listen () initialize port binding and connection queues.**
* **Channel.register (selector, OP _ ACCEPT) registers events only inside the Selector and does not directly call the kernel epoul _ ctl;**
* **The real fd registration occurs on the next selector.select () call, when epol _ ctl (ADD / MOD) is called via JNI to encapsulate fd as epitem into the eventpoll red-black tree.**
* **Select calls epol _ wait () to get the ready event and map it to SelectionKeyImpl back to the Java application layer.**

4. **From kernel to user state**

* **When the user thread is awakened, it proceeds from the chokepoint, fetching events from the event poll's rdllist and handing them over to the application.**
* **In this process, network events complete the life cycle from electrical signal → kernel → socket → epol → selector → application event cycle.**
