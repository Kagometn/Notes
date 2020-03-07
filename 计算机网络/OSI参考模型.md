   

### OSI参考模型

#### 分层结构：

##### 为什么实现分成结构：

1）**各层之间相互独立**：高层是不需要知道底层的功能是采取硬件技术来实现的，它只需要知道通过与底层的接口就可以获得所需要的服务；

 

2）**灵活性好**：各层都可以采用最适当的技术来实现，例如某一层的实现技术发生了变化，用硬件代替了软件，只要这一层的功能与接口保持不变，实现技术的变化都并不会对其他各层以及整个系统的工作产生影响； 



3）**易于实现和标准化**：由于采取了规范的层次结构去组织网络功能与协议，因此可以将计算机网络复杂的通信过程，划分为有序的连续动作与有序的交互过程，有利于将网络复杂的通信工作过程化解为一系列可以控制和实现的功能模块，使得复杂的计算机网络系统变得易于设计，实现和标准化



为了解决计算机网络中复杂的大问题，从而**按功能**引出了**分层结构**

目的：支持**异构网络系统**的互联互通

![image-20191122181058559](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122181058559.png)

![image-20191122184335371](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184335371.png)

![image-20191122184417718](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184417718.png)



#### 各层功能：

##### 应用层：

![image-20191122184535207](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184535207.png)

##### 表示层：

**在传输前是否需要进行加密或者压缩处理，二进制ASCII码**

![image-20191122184616539](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184616539.png)



##### 会话层：

**查看木马(netStat -n)**

![image-20191122184724580](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184724580.png)

##### 传输层：

![image-20191122184836736](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184836736.png)

![image-20191122184859655](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122184859655.png)

##### 网络层：

![image-20191122175548073](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122175548073.png)

##### 数据链路层：

![image-20191122175724480](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122175724480.png)

##### 物理层：

![image-20191122175941344](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20191122175941344.png)

****

#### OSI参考模型对网络排序的指导

当网络出现错误的时候应该从下往上排查，即先从

物理层：查看连接状态，发送和接受的数据包，网线是否连接

数据链路层：

- **MAC地址冲突**：MAC地址虽然是全球唯一的，但是可以通过修改注册表更改本机的MAC地址(网络连接完好，一个交换机可以接受的MAC地址不可以重复)

- **ADSL欠费**

- **网速没办法协商一致**，即服务器和交换机之间的带块之间无法协商

- **计算机连接到错误的VLAN**

  

网络层：规划地址，选择路径

- 配置错误的IP地址  子网掩码 
- 配置错误的网关
- 路由器没有到达指定的目标地址

应用层故障：

- 应用程序配置错误



#### OSI参考模型以及网络安全

物理层安全：他人可以随意接入你的网络；将用不到的网线将其从交换器拔掉或者是shutdown

数据链路层：

- ADSL
- 账号密码
- VLAN  
- 交换机端口绑定MAC地址

网络层：

- 在路由器上使用ACL控制数据包流量；
- 设置高级windows防火墙

应用层

- 开发的程序没有漏洞

#### TCP/IP协议和OSI参考模型

![image-20200209184407976](C:\Users\Lenovo\Desktop\计算机网络\image-20200209184407976.png)

ipv4:美国国防部为了战争而开发的网络，没有链接全球的网络一起



**装箱：**

**应用层**：准备要传输的数据

**传输层**：有余数据太大，传输层将数据分段，编号

**网络层**：负责写地址：原地址以及目标地址；加上目标MAC地址以及源MAC地址，路由器根据IP地址选择路径

![image-20200209185207048](C:\Users\Lenovo\Desktop\计算机网络\image-20200209185207048.png)





![image-20200209185435036](C:\Users\Lenovo\Desktop\计算机网络\image-20200209185435036.png)

FCS:

**拆箱：**

反向操作



![image-20200209185538992](C:\Users\Lenovo\Desktop\计算机网络\image-20200209185538992.png)

![image-20200209185603649](C:\Users\Lenovo\Desktop\计算机网络\image-20200209185603649.png)



**对等透明**



计算机 网络的性能指标

![image-20200209185713691](C:\Users\Lenovo\Desktop\计算机网络\image-20200209185713691.png)



**速率：**

![image-20200209185738674](C:\Users\Lenovo\Desktop\计算机网络\image-20200209185738674.png)

- **比特率：**计算机传输的数据都是二进制的形式，一个0/1是一个比特，而比特率就是单位时间内传送数据的多少
- **信道：**发送端到接受端的道路

**带宽：**

![image-20200209190243634](C:\Users\Lenovo\Desktop\计算机网络\image-20200209190243634.png)

**吞吐量：**

![image-20200209190543630](C:\Users\Lenovo\Desktop\计算机网络\image-20200209190543630.png)

**时延**

![image-20200209190644478](C:\Users\Lenovo\Desktop\计算机网络\image-20200209190644478.png)

- 发送时延：从开始发送到数据完全发送结束
- 传播时延：（**丢包**）
- 处理时延：
- 排队时延：

**时延带宽积**：在传输的时候有多少数据

![image-20200209190956380](C:\Users\Lenovo\Desktop\计算机网络\image-20200209190956380.png)



**往返时间 ** 			**PING**:往返时间  (>2000ms时显示超时)

![image-20200209191102953](C:\Users\Lenovo\Desktop\计算机网络\image-20200209191102953.png)

**利用率**

![image-20200209191254068](C:\Users\Lenovo\Desktop\计算机网络\image-20200209191254068.png)



