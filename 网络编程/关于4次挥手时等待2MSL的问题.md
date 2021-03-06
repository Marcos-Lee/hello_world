# 关于4次挥手时等待2MSL的问题
关于三次握手和四次挥手的过程算是十分熟悉了，可昨天网易面试面试官的一个问题瞬间让我意识到自己只不过理解了一点皮毛而已。

我们都知道，四次挥手时，主动发起关闭连接的操作的一方将达到TIME_WAIT状态，而且这个状态要保持Maximum Segment Lifetime的两倍时间。


1.为什么不直接关闭要进入等待状态？  
2. 为什么等待的时间是2MSL？

一、为什么不直接关闭要进入等待状态？

1.保证客户端发送的ACK报文段能够到达服务器，从而保证tcp连接能够进行可靠的关闭。

 这个很好理解，如果客户端发动ACK后就立刻关闭，那么如果ACK丢失的话，服务端就会一直处于等待关闭确认的状态，超时后再发送关闭请求时，此时的客户端已经关闭，那么服务端就无法进行正常的关闭。

2.保证此次连接的数据段消失，防止失效的数据段。

客户端在发送ACK后，再等待2MSL时间，可以使本次连接所产生的数据段从网络中消失，从而保证关闭连接后不会有还在网络中滞留的数据段去骚扰服务端；

再有一点就是，如果客户端重新发送请求，如果前一次连接的某些数据仍然滞留在网络中，这些延迟数据在建立新连接之后才到达Server，由于新连接和老连接的端口号是一样的，又因为TCP协议判断不同连接的依据是socket pair，于是，TCP协议就认为那个延迟的数据是属于新连接的，这样就和真正的新连接的数据包发生混淆了。

二、为什么是2MSL

我们知道服务端收到ACK，关闭连接。但是客户端无法知道ACK是否已经到达服务端，于是开始等待？等待什么呢？假如ACK没有到达服务端，服务端会为FIN这个消息超时重传 timeout retransmit ，那如果客户端等待时间足够，又收到FIN消息，说明ACK没有到达服务端，于是再发送ACK，直到在足够的时间内没有收到FIN，说明ACK成功到达。这个等待时间至少是：服务端的timeout + FIN的传输时间，为了保证可靠，采用更加保守的等待时间2MSL。

MSL，Maximum Segment Life，这是TCP 对TCP Segment 生存时间的限制。

客户端发出ACK，等待ACK到达对方的超时时间 MSL，等待FIN的超时重传，也是MSL，所以如果2MSL时间内没有收到FIN，说明对方安全收到FIN。
