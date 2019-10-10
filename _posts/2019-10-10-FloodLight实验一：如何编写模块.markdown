---
layout:     post
title:      FloodLight实验一：如何编写模块
subtitle:   Experiment of FloodLight
author:     "Ashior"
header-img: "img/home-bg-science.jpg"
mathjax:    true
catalog:     true
tags:
  - SDN
  - FloodLight
---


# 实验一：如何编写模块

## 先决条件

实验目的：我们将创建一个捆绑包，该捆绑包将监视以前从未见过的新MAC地址，并记录MAC并将其打开。
注意，再开始本教程之前，你应该成功完成"入门"教程，包括设置Eclipse、已安装并正在运行Mininet或物理OpenFlow交换机

## 创建监听器

在Eclipse中添加类
1. 在`Package Explorer`中展开`floodlight`项，然后找到"src/main/java"文件夹。
2. 右键单击`src/main/java`文件夹，然后选择"新建/类"。
3. 在`包`框中输入`net.floodlightcontroller.mactracker`。
4. 在"名称"框中输入`MACTracker`。
5. 在"接口"框旁边，选择`添加...`。
6. 添加`IOFMessageListener`和`IFloodlightModule`，单击`确定`。
7. 在对话框中单击`完成`。

完成后，你可以看见以下源代码：

```java
package net.floodlightcontroller.mactracker;
 
import java.util.Collection;
import java.util.Map;
 
import org.projectfloodlight.openflow.protocol.OFMessage;
import org.projectfloodlight.openflow.protocol.OFType;
import org.projectfloodlight.openflow.types.MacAddress;
 
import net.floodlightcontroller.core.FloodlightContext;
import net.floodlightcontroller.core.IOFMessageListener;
import net.floodlightcontroller.core.IOFSwitch;
import net.floodlightcontroller.core.module.FloodlightModuleContext;
import net.floodlightcontroller.core.module.FloodlightModuleException;
import net.floodlightcontroller.core.module.IFloodlightModule;
import net.floodlightcontroller.core.module.IFloodlightService;
 
public class MACTracker implements IOFMessageListener, IFloodlightModule {
 
    @Override
    public String getName() {
        // TODO Auto-generated method stub
        return null;
    }
 
    @Override
    public boolean isCallbackOrderingPrereq(OFType type, String name) {
        // TODO Auto-generated method stub
        return false;
    }
 
    @Override
    public boolean isCallbackOrderingPostreq(OFType type, String name) {
        // TODO Auto-generated method stub
        return false;
    }
 
    @Override
    public Collection<Class<? extends IFloodlightService>> getModuleServices() {
        // TODO Auto-generated method stub
        return null;
    }
 
    @Override
    public Map<Class<? extends IFloodlightService>, IFloodlightService> getServiceImpls() {
        // TODO Auto-generated method stub
        return null;
    }
 
    @Override
    public Collection<Class<? extends IFloodlightService>> getModuleDependencies() {
        // TODO Auto-generated method stub
        return null;
    }
 
    @Override
    public void init(FloodlightModuleContext context)
            throws FloodlightModuleException {
        // TODO Auto-generated method stub
 
    }
 
    @Override
    public void startUp(FloodlightModuleContext context) {
        // TODO Auto-generated method stub
 
    }
 
    @Override
    public Command receive(IOFSwitch sw, OFMessage msg, FloodlightContext cntx) {
        // TODO Auto-generated method stub
        return null;
    }
 
}
```

## 设置模块依赖关系和初始化

在开始之前，我们将需要一些依赖关系才能使代码正常工作。像Eclipse这样的工具应该使添加它们变得容易。但是，如果您不使用Eclipse，则可能只想在前面添加它们：

```java
import net.floodlightcontroller.core.IFloodlightProviderService;
import java.util.ArrayList;
import java.util.concurrent.ConcurrentSkipListSet;
import java.util.Set;
import net.floodlightcontroller.packet.Ethernet;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

现在我们有了骨架类，我们必须实现正确的功能以使模块可加载。让我们从在类中注册一些我们需要的成员变量开始。由于我们正在监听`OpenFlow`消息，因此需要向`FloodlightProvider`注册（`IFloodlightProviderService`类）。我们还需要一个集合来存储我们见过的`macAddress`。最后，我们需要一个记录器来输出我们所看到的。

```java
protected IFloodlightProviderService floodlightProvider;
protected Set<Long> macAddresses;
protected static Logger logger;
```


现在我们需要将其连接到模块加载系统。我们通过修改`getModuleDependencies()`函数来告诉模块加载器我们依赖它。

```java
@Override
public Collection<Class<? extends IFloodlightService>> getModuleDependencies() {
    Collection<Class<? extends IFloodlightService>> l =
        new ArrayList<Class<? extends IFloodlightService>>();
    l.add(IFloodlightProviderService.class);
    return l;
}
```

现在该创建我们的`init`方法了。`init`在控制器启动过程的早期称呼，它主要用于加载依赖项和初始化数据结构。

```java
@Override
public void init(FloodlightModuleContext context) throws FloodlightModuleException {
    floodlightProvider = context.getServiceImpl(IFloodlightProviderService.class);
    macAddresses = new ConcurrentSkipListSet<Long>();
    logger = LoggerFactory.getLogger(MACTracker.class);
}
```

## 处理入站消息

现在该实现基本的侦听器了。我们将在我们的启动方法中注册`PACKET_IN`消息。在这里，我们可以确保我们依赖的其他模块已经初始化。

```java
@Override
public void startUp(FloodlightModuleContext context) {
    floodlightProvider.addOFMessageListener(OFType.PACKET_IN, this);
}
```
我们还必须为我们的`OFMessage`侦听器输入一个ID。这是在`getName()`调用中完成的。

```java
@Override
public String getName() {
    return MACTracker.class.getSimpleName();
}
```

现在，我们必须为`PACKET_IN`消息定义所需的行为。请注意，我们返回`Command.CONTINUE`以允许此消息继续由其他`PACKET_IN`处理程序处理。

```java
@Override
public net.floodlightcontroller.core.IListener.Command receive(IOFSwitch sw, OFMessage msg, FloodlightContext cntx) {
    Ethernet eth =
            IFloodlightProviderService.bcStore.get(cntx,
                                        IFloodlightProviderService.CONTEXT_PI_PAYLOAD);

    Long sourceMACHash = eth.getSourceMACAddress().getLong();
    if (!macAddresses.contains(sourceMACHash)) {
        macAddresses.add(sourceMACHash);
        logger.info("MAC Address: {} seen on switch: {}",
                eth.getSourceMACAddress().toString(),
                sw.getId().toString());
    }
    return Command.CONTINUE;
}
```

如果您想了解有关如何检查IP，TCP等高层报文头的更多信息，请[参考本教程](https://floodlight.atlassian.net/wiki/spaces/floodlightcontroller/pages/9142279/How+to+Process+a+Packet-In+Message)。


## 处理OpenFlow消息时的订购模块

尽管在本教程中不是必需的，但通常有必要定义`IOFMessageListeners`处理`OpenFlow`消息的顺序。`IOFMessageListener`的`isCallbackOrderingPrereq`（OFType类型，字符串名称）和`isCallbackOrderingPostreq`（OFType类型，字符串名称）函数定义处理从交换机接收的数据包的模块顺序。一次接收到的每个`OpenFlow`消息（例如`PACKET_IN`）都由一个模块进行处理，以便模块可以将有关该数据包的元数据从一个传递到另一个，或者完全终止处理链。`isCallbackOrderingPrereq()`定义在处理特定类型的`OpenFlow`消息时应在我们的模块之前运行的模块。`isCallbackOrderingPostreq()`定义在处理特定类型的`OpenFlow`消息时应在我们的模块之后运行的模块。一种常见的用法是告诉实现第二层反应式数据包转发的转发模块在我们的模块之后运行，因为转发将插入流并修改网络状态。**如果要插入自己的算法定义的流**，则需要告诉`Forwarding`在模块之后运行，因此应使用`isCallbackOrderingPostreq()`。

例如，设备管理器对`PACKET_IN`消息作出反应，以了解主机在网络中的位置。转发依赖于设备管理器了解的设备，以将流从点`A`插入到网络中的点`B`。因此，转发需要告诉设备管理器先处理`PACKET_IN`消息。看一下`Forwarding`的源代码，可以发现`Forwarding`是[如何做到这一点的](https://floodlight.atlassian.net/wiki/spaces/floodlightcontroller/pages/9142279/How+to+Process+a+Packet-In+Message)。

特别注意，另一个模块的名称作为参数传递给两个回调排序函数中的任何一个。这是我们应该为同样作为参数传入的`OpenFlow`消息类型做出决定的模块。我们的决定通过返回`true`（是）或`false`（否）来反映。每个模块的名称由`IOFMessageListener`的`getName()`定义，[如此处所示](https://github.com/floodlight/floodlight/blob/master/src/main/java/net/floodlightcontroller/routing/ForwardingBase.java#L142)和上面所讨论的。

## 注册模块

我们差不多完成了，现在我们只需要告诉Floodlight在启动时加载模块即可。首先，我们必须告诉加载程序该模块存在。这是通过在`src/main/resources/META-INF/services/net.floodlightcontroller.core.module.IFloodlightModule`自己的行上添加完全限定的模块名称来完成的。我们打开该文件并添加以下行：

```java
net.floodlightcontroller.mactracker.MACTracker
```

然后，我们告诉要加载的模块。我们修改Floodlight模块配置文件以附加`MACTracker`。默认值是`src/main/resources/floodlightdefault.properties`。关键字是`Floodlight.modules`，值是逗号分隔的标准模块名称列表。

```java
floodlight.modules = <leave the default list of modules in place>, net.floodlightcontroller.mactracker.MACTracker
```

最后，通过右键单击Main.java 并选择`Run As .../Java Application`来运行控制器。

## 如何将Mininet软件OpenFlow交换机连接到Floodlight

假设您在主机上的VM内运行`Mininet`，并且在主机上从`Eclipse`运行`Floodlight`。确定主机相对于`Mininet`的`IP`地址，在以下示例中，将其设置为网关（192.168.110.2）

```java
mininet@mininet:~$ sudo route -n
Kernel IP routing table
Destination Gateway Genmask Flags Metric Ref Use Iface
192.168.110.0 0.0.0.0 255.255.255.0 U 0 0 0 eth0
0.0.0.0 192.168.110.2 0.0.0.0 UG 0 0 0 eth0
```

```java
mininet@mininet:~$ sudo mn --controller=remote,ip=192.168.110.2,port=6653 --switch=ovsk,protocols=OpenFlow13
*** Loading ofdatapath
*** Adding controller
*** Creating network
*** Adding hosts:
h2 h3
*** Adding switches:
s1
*** Adding edges:
(s1, h2) (s1, h3)
*** Configuring hosts
h2 h3
*** Starting controller
*** Starting 1 switches
s1
*** Starting CLI:
mininet>pingall
```

Pingall命令应从控制台上的MACTracker生成调试输出。

----

# 实验过程

## 小结

我们创建了包：`net.floodlightcontroller.mactracker`，并在里面创建了`MACTracker.java`文件。之后在`src/main/resources/META-INF/services/net.floodlightcontroller.core.module.IFloodlightModule`中添加我们的模块：`net.floodlightcontroller.mactracker.MACTracker`最后在`src/main/resources/floodlightdefault.properties`关键字是`Floodlight.modules`中添加我们的模块：`floodlight.modules = <leave the default list of modules in place>, net.floodlightcontroller.mactracker.MACTracker`。

## 实验结果

当你完成以上步骤时，你将获得以下文件：

```java
package net.floodlightcontroller.mactracker;

import java.util.Collection;

/*
 * 我们将创建一个捆绑包，它将监视以前从未见过的新MAC地址，并记录它们被看到的MAC和交换机。
 * 我们差不多完成了，现在我们只需告诉Floodlight在启动时加载模块。
 * 首先，我们必须告诉加载器模块是否存在。
 * 这是通过在src/main/resources/META-INF/services/
 * net.floodlightcontroller.core.module.IFloodlightModule
 * 中在其自己的行上添加完全限定的模块名称来完成的。我们打开该文件并附加以下行：
 * net.floodlightcontroller.mactracker.MACTracker
 * 然后我们告诉模块加载。我们修改Floodlight模块配置文件以附加MACTracker。
 * 默认值为src/main/resources/floodlightdefault.properties。
 * 关键是floodlight.modules，值是逗号分隔的完全限定模块名称列表。
 * floodlight.modules = <leave the default list of modules in place>, net.floodlightcontroller.mactracker.MACTracker
 */


public class MACTracker implements IOFMessageListener, IFloodlightModule {
	/*
	 * 我们必须实现正确的函数来使模块可加载。
	 * 让我们首先将一些我们需要的成员变量注册到类中。
	 * 由于我们正在收听OpenFlow消息，
	 * 因此我们需要向FloodlightProvider（IFloodlightProviderService类）注册。
	 * 我们还需要一个存储我们见过的macAddresses的集合。
	 * 最后，我们需要一个记录器来输出我们所看到的内容。
	 */
	protected IFloodlightProviderService floodlightProvider;
	protected Set<Long> macAddresses;
	protected static Logger logger;

//	我们还必须为OFMessage监听器输入一个ID。这是在getName（）调用中完成的。
	@Override
	public String getName() {
		// TODO Auto-generated method stub
//		return null;
		return MACTracker.class.getSimpleName();
	}
//	isCallbackOrderingPrereq()定义在处理特定类型的OpenFlow消息时应该在模块之前运行的模块。
	@Override
	public boolean isCallbackOrderingPrereq(OFType type, String name) {
		// TODO Auto-generated method stub
		return false;
	}
//	isCallbackOrderingPostreq()定义在处理特定类型的OpenFlow消息时应该在我们的模块之后运行的模块。
	@Override
	public boolean isCallbackOrderingPostreq(OFType type, String name) {
		// TODO Auto-generated method stub
		return false;
	}
/*
 * 一个常见的用途是告诉Forwarding模块，它实现了第2层反应式数据包转发，
 * 在我们的模块之后运行，因为Forwarding将插入流并修改网络状态。
 * 如果我们想要插入由我们自己的算法定义的自己的流，
 * 那么我们需要告诉Forwarding在我们的模块之后运行，
 * 因此我们应该使用isCallbackOrderingPostreq()。
 */
	@Override
	public Collection<Class<? extends IFloodlightService>> getModuleServices() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	public Map<Class<? extends IFloodlightService>, IFloodlightService> getServiceImpls() {
		// TODO Auto-generated method stub
		return null;
	}
//	现在我们需要将它连接到模块加载系统。
//	我们通过修改getModuleDependencies()函数告诉我们依赖它的模块加载器。
	@Override
	public Collection<Class<? extends IFloodlightService>> getModuleDependencies() {
		// TODO Auto-generated method stub
//		return null;
		Collection<Class<? extends IFloodlightService>> l =
		        new ArrayList<Class<? extends IFloodlightService>>();
	    l.add(IFloodlightProviderService.class);
	    return l;
	}
	
//	Init在控制器启动过程的早期被调用 - 它主要用于加载依赖项并初始化数据结构。
	@Override
	public void init(FloodlightModuleContext context) throws FloodlightModuleException {
		// TODO Auto-generated method stub
		floodlightProvider = context.getServiceImpl(IFloodlightProviderService.class);
	    macAddresses = new ConcurrentSkipListSet<Long>();
	    logger = LoggerFactory.getLogger(MACTracker.class);
	}
//	实现基本的监听器了。我们将在startUp方法中注册PACKET_IN消息。
	@Override
	public void startUp(FloodlightModuleContext context) throws FloodlightModuleException {
		// TODO Auto-generated method stub
		floodlightProvider.addOFMessageListener(OFType.PACKET_IN, this);
	}
//	现在我们必须为PACKET_IN消息定义我们想要的行为。
//	请注意，我们返回Command.CONTINUE以允许此消息继续由其他PACKET_IN处理程序处理。
	@Override
	public Command receive(IOFSwitch sw, OFMessage msg, FloodlightContext cntx) {
		// TODO Auto-generated method stub
//		return null;
		Ethernet eth =
                IFloodlightProviderService.bcStore.get(cntx,
                                            IFloodlightProviderService.CONTEXT_PI_PAYLOAD);
 
        Long sourceMACHash = eth.getSourceMACAddress().getLong();
        if (!macAddresses.contains(sourceMACHash)) {
            macAddresses.add(sourceMACHash);
            logger.info("YELLOW LOGGER: MAC Address: {} seen on switch: {}",
                    eth.getSourceMACAddress().toString(),
                    sw.getId().toString());
        }
        return Command.CONTINUE;
	}
	
}
```

在`Mininet`中输入命令：

```vim
sudo mn --controller=remote,ip=127.0.0.1,port=6653 --switch=ovsk,protocols=OpenFlow13
```

执行后可以看下如下图的效果：

![1](https://pic4.zhimg.com/80/v2-50a96d13cfa4c7fdfe5c434056110dbb_hd.jpg)

这是创建了一台交换机，连接两台主机的网络拓扑。

打开Firefox，输入网址：`127.0.0.1:8080/ui/pages/index.html`

![2](https://pic4.zhimg.com/80/v2-b0516d2b80261028ad97d38b24a0edd7_hd.jpg)

可以看到在交换机与主机的地方显示了已经连接的数量。

![3](https://pic2.zhimg.com/80/v2-1d8e6ee70fbf9a67274a2b5b1cfb56cd_hd.jpg)

点击上面的Topology就可以看到图形化的网络拓扑结构。之后再Mininet中输入命令`pingall`后，再`shell`中可以看到ping通两个主机，同时在Eclipse中出现我们所新增的模块功能。

![4](https://pic3.zhimg.com/80/v2-766b7b23ce672eda49c9124963247d5e_hd.jpg)

![5](https://pic1.zhimg.com/80/v2-aa8d5251dcb52edbc5ac41499626161c_hd.jpg)

以上就是对官方教程的实验过程。

