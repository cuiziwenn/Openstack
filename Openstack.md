正文

一、openstack由来 　　
openstack最早由美国国家航空航天局NASA研发的Nova和Rackspace研发的swift组成。后来以apache许可证授权,旨在为公共及私有云平台建设。openstack主要用来为企业内部实现类似于Amazon EC2和S3的云基础架构服务（Iaas）.每6个月更新一次，基本与ubuntu同步，命名是以A-Z作为首字母来的。

二、openstack项目与组件（服务名是项目名的别名）

1、核心项目3个 （1）控制台 服务名：Dashboard

项目名：Horizon

功能：web方式管理云平台，建云主机，分配网络，配安全组，加云盘。
（2）计算 服务名：计算

项目名：Nova（可以支持各种各样的虚拟化技术，vmware\kvm等）

功能：负责响应虚拟机创建请求、调度、销毁云主机。
（3）网络 服务名：网络

项目名：Neutron（实现网络虚拟化）

功能：实现SDN（软件定义网络），提供一整套API，用户可以基于该API实现自己定义专属网络，不同厂商可以基于此API提供自己的产品实现。
2、共享服务项目3个 （1）认证服务 服务名：认证服务

项目名：Keystone

功能：为访问openstack各组件提供认证和授权功能，认证通过后，提供一个服务列表（存放你有权访问的服务），可以通过该列表访问各个组件。
（2）镜像服务 服务名：镜像服务

项目名：Glance

功能：为云主机安装操作系统提供不同的镜像选择
（3）计费服务 服务名：计费服务

项目名：Ceilometer（监控）

功能：收集云平台资源使用数据，用来计费或者性能监控
3、存储项目2个（附加项目） 　　现在主流的存储主要是三种：文件存储、块存储、对象存储。

　　文件存储相当于一个大的文件夹，典型是FTP\NFS服务器，以文件作为传输协议。Ext3、Ext4、NTFS是本地文件存储，NFS、CIFS是网络文件存储（NAS存储）；最明显的特征是支持POSIX的文件访问接口：open、read、write、seek、close等；优点：便于扩展&共享；缺点：读写速度慢。

　　块存储在物理级别的最小读写单位是扇区。块存储可以认为是裸盘，最多包一层逻辑卷（LVM）；常见的DAS、FC-SAN、IP-SAN都是块存储，块存储最明显的特征就是不能被操作系统直接读写，需要格式化为指定的文件系统（Ext3、Ext4、NTFS）后才可以访问。优点：读写快（带宽&IOPS）；缺点：因为太底层了，不利于扩展。

　　对象存储将元数据独立了出来，控制节点叫元数据服务器（服务器+对象存储管理软件），里面主要负责存储对象的属性（主要是对象的数据被打散存放到了那几台分布式服务器中的信息），而其他负责存储数据的分布式服务器叫做OSD，主要负责存储文件的数据部分。当用户访问对象，会先访问元数据服务器，元数据服务器只负责反馈对象存储在哪些OSD，假设反馈文件A存储在B、C、D三台OSD，那么用户就会再次直接访问3台OSD服务器去读取数据。

　　基于rest api的方式访问，说穿了就是url地址。对象存储和分布式文件系统的表面区别：对象存储支持的访问接口基本都是restful接口、而分布式文件系统提供的POSIX兼容的文件操作接口；

（1）对象存储 服务名：对象存储

项目名：Swift

功能：REST风格的接口和扁平的数据组织结构。RESTFUL HTTP API来保存和访问任意非结构化数据，ring环的方式实现数据自动复制和高度可以扩展架构，保证数据的高度容错和可靠性
（2）块存储 服务名：块存储

项目名：Cinder

功能：提供持久化块存储，即为云主机提供附加云盘。
4、高层服务项目1个 （1）编排服务 服务名：编排服务

项目名：Heat

功能：自动化部署应用，自动化管理应用的整个生命周期.主要用于Paas 
三、openstack各组件详解及运行流程

1、各组件逻辑关系图 　　
https://github.com/cuiziwenn/Openstack/blob/main/Openstck%E8%BF%90%E8%A1%8C%E6%B5%81%E7%A8%8B.png
2、openstack新建云主机流程图
https://github.com/cuiziwenn/Openstack/blob/main/Openstack%E6%96%B0%E5%BB%BA%E4%B8%BB%E6%9C%BA%E6%B5%81%E7%A8%8B.png
3、虚拟机启动过程 
1.界面或命令行通过RESTful API向keystone获取认证信息。

2.keystone通过用户请求认证信息，并生成auth-token返回给对应的认证请求。

3.界面或命令行通过RESTful API向nova-api发送一个boot instance的请求（携带auth-token）。

4.nova-api接受请求后向keystone发送认证请求，查看token是否为有效用户和token。

5.keystone验证token是否有效，如有效则返回有效的认证和对应的角色（注：有些操作需要有角色权限才能操作）。

6.通过认证后nova-api和数据库通讯。

7.初始化新建虚拟机的数据库记录。

8.nova-api通过rpc.call向nova-scheduler请求是否有创建虚拟机的资源(Host ID)。

9.nova-scheduler进程侦听消息队列，获取nova-api的请求。

10.nova-scheduler通过查询nova数据库中计算资源的情况，并通过调度算法计算符合虚拟机创建需要的主机。

11.对于有符合虚拟机创建的主机，nova-scheduler更新数据库中虚拟机对应的物理主机信息。

12.nova-scheduler通过rpc.cast向nova-compute发送对应的创建虚拟机请求的消息。

13.nova-compute会从对应的消息队列中获取创建虚拟机请求的消息。

14.nova-compute通过rpc.call向nova-conductor请求获取虚拟机消息。（Flavor）

15.nova-conductor从消息队队列中拿到nova-compute请求消息。

16.nova-conductor根据消息查询虚拟机对应的信息。

17.nova-conductor从数据库中获得虚拟机对应信息。

18.nova-conductor把虚拟机信息通过消息的方式发送到消息队列中。

19.nova-compute从对应的消息队列中获取虚拟机信息消息。

20.nova-compute通过keystone的RESTfull API拿到认证的token，并通过HTTP请求glance-api获取创建虚拟机所需要镜像。

21.glance-api向keystone认证token是否有效，并返回验证结果。

22.token验证通过，nova-compute获得虚拟机镜像信息(URL)。

23.nova-compute通过keystone的RESTfull API拿到认证k的token，并通过HTTP请求neutron-server获取创建虚拟机所需要的网络信息。

24.neutron-server向keystone认证token是否有效，并返回验证结果。

25.token验证通过，nova-compute获得虚拟机网络信息。

26.nova-compute通过keystone的RESTfull API拿到认证的token，并通过HTTP请求cinder-api获取创建虚拟机所需要的持久化存储信息。

27.cinder-api向keystone认证token是否有效，并返回验证结果。

28.token验证通过，nova-compute获得虚拟机持久化存储信息。

29.nova-compute根据instance的信息调用配置的虚拟化驱动来创建虚拟机。
