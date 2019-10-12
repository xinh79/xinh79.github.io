---
layout:     post
title:      FloodLight实验二：如何使用OpenFlowJ-Loxigen
subtitle:   OpenFlowJ-Loxigen of FloodLight
author:     "Ashior"
header-img: "img/home-bg-science.jpg"
mathjax:    true
catalog:     true
tags:
  - SDN
  - FloodLight
---


## 背景

新的`OpenFlowJ-Loxigen`库支持`Floodlight v1.0`及更高版本。`OpenFlowJ-Loxigen`支持从`1.0`到`1.5`的多个`OpenFlow`版本。通过单个通用的版本无关的API，`Floodlight v1.2`最多支持`1.4`，而`master`分支最多支持`v1.5`。开发人员可以利用API编写与多个和各种`OpenFlow`版本的交换机兼容的应用程序。可从`OpenFlowJ-Loxigen`库访问所有`OpenFlow`概念和类型。这个想法是每个`OpenFlow`版本都有一个工厂，该**工厂可以构建针对该OpenFlow版本定义的所有类型和消息**。

`OpenFlowJ-Loxigen`还采用了一种经过改进的新方法来创建`OpenFlow`消息，匹配，动作，FlowMod等。使用构建器可以大大简化许多`OpenFlow`对象的创建过程，所有构建器都可以从常见的`OpenFlow`工厂访问。只需设置您的字段并构建。诸如消息长度通配符之类的底层细节的处理在后台进行。您再也不必担心跟踪和正确设置消息长度。编写`FlowMod`时，所有Match字段都会自动通配符，但您在`FlowMod`中专门设置的字段除外。还支持屏蔽字段，并且可以使用"匹配"指定屏蔽，如果需要，它将自动设置适当的通配符位。由生成器生成的所有对象都是不可变的，这使代码更安全，并使您的应用程序更易于调试。

连接到`Floodlight`的所有交换机都包含一个适用于该交换机所使用的`OpenFlow`版本的工厂。可能有多个交换机，所有交换机都使用不同版本的`OpenFlow`，其中`OpenFlowJ-Loxigen`处理后台的底层协议差异。从模块和应用程序开发人员的角度来看，该开关只是作为`IOFSwitch`公开，它具有功能`getOFFactory()`以返回适用于该开关所讲的OpenFlow版本的`OpenFlowJ-Loxigen`工厂。一旦有了正确的工厂，就可以通过通用`API OpenFlowJ-Loxi`公开创建OpenFlow类型和概念。

因此，在编写`FlowMod`和其他类型时，您无需切换API。假设您希望构建一个FlowMod并将其发送到交换机。交换机管理器已知的每个交换机都有对相同版本的`OpenFlow`工厂的引用，该工厂在交换机和控制器之间的初始握手中协商过。只需从您的交换机引用工厂，创建构建器，构建`FlowMod`，然后将其写入交换机即可。无论所有OpenFlow版本如何，都为所有`OpenFlow`对象的构造公开相同的API。但是，您将需要知道每个`OpenFlow`版本都可以执行的操作。否则，例如，如果您告诉`OF1.0`开关执行某些操作，例如添加`Group`(其`OpenFlow`版本不支持)，为了更好，`OpenFlowJ-Loxigen`还引入了其他一些细微的变化。例如，`OpenFlowJ-Loxi`库分别通过`DatapathId`，`OFPort`，`IPv4Address`和`MacAddress`定义了许多常见类型，例如交换机数据路径ID，`OpenFlow`端口以及IP和MAC地址。鼓励您探索`org.projectfloodlight.openflow.types`，在这里您会发现各种各样的通用类型，这些通用类型现在可以在一个位置方便地定义。就像上面的构建器生成的对象一样，所有类型都是不可变的。

最后，`OpenFlowJ-Loxigen`是开放源代码，具有一整套自动生成的协议单元测试套件，一个经过集成测试的自动化发布过程，并且被当前的商用BSN系列产品用于生产。

## 先决条件和目标

本教程的目的是介绍用户模块内`OpenFlowJ-Loxigen`库的最常见用法。给出了有关如何执行日常操作(例如编写FlowMod，处理PacketIn等)的示例。这些示例介绍了这些技术，并假定您编写了可以在其中应用它们的模块。如果您想了解如何为`Floodlight v1.0`创建模块，请参[阅本教程](https://xinh79.github.io/2019/10/10/FloodLight%E5%AE%9E%E9%AA%8C%E4%B8%80-%E5%A6%82%E4%BD%95%E7%BC%96%E5%86%99%E6%A8%A1%E5%9D%97/)。

## OpenFlow概念

几乎每个OpenFlow概念(OFObject，例如Match，OFAction，OFMessage等)都是通过构建器构建的，所有构建器都通过OFFactory接口公开。由于不同OpenFlow版本之间OpenFlow协议的差异，因此每个OpenFlow版本都有一个特定的工厂：

- OFFactoryVer10
- OFFactoryVer11
- OFFactoryVer12
- OFFactoryVer13
- OFFactoryVer14
- OFFactoryVer15

这些都实现了OFFactory接口，因此无论工厂的特定版本如何，您都可以在代码中简单地使用OFFactory接口。

有很多方法可以获取所需的OFFactory的引用。您可以通过指定OFVersion枚举从`OpenFlowJ-Loxigen`要求特定的OFFactory版本：

```java
OFFactory my13Factory = OFFactories.getOFFactory(OFVersion.OF_13);
/* Get an OpenFlow 1.3 factory. */
```

也许更有用，您可以从`IOFSwitch`获得工厂(有关如何获取`v1.0`版的`Floodlight`模块中有关如何获取`IOFSwitch`对象的更多信息)。

```java
IOFSwitch mySwitch = switchService.getSwitch(DatapathId.of("00:00:00:00:00:00:00:01"));
OFFactory myFactory = mySwitch.getOFFactory();
/* Use the factory version appropriate for the switch in question. */
```

您还可以从由`OFFactory`本身生成的现有对象中获取特定的`OFFactory`。从`OFFactory`构造的任何对象都设置有与构造它的`OFFactory`相同的`OFVersion`。

```java
OFVersion flowModVersion = myFlowMod.getVersion();
/* We assume myFlowMod has been already constructed (keep reading for more info on building objects) */
OFFactory myFactory = OFFactories.getFactory(flowModVersion);
/* Get the OFFactory version we need based on the existing object's version. */
```

每个`OFFactory`版本的`API`都是相同的，这意味着作为开发人员，您需要了解使用`OFFactory`的`OpenFlow`版本的功能。对于所有`OFFactory`版本，`OFFactory`所属的`OpenFlow`版本不支持的`OFFactory`中的任何操作都将引发`UnsupportedOperationException`。例如，如果您尝试从`OFVersion.OF_10`的`OFFactory`生成`OFGroupAdd`，则可以尝试，但是由于`OpenFlow 1.0`不支持`Groups`，因此`OFFactory`将抛出`UnsupportedOperationExection`：

`OFGroupAdd not supported in version 1.0`

一个好的做法是，首先确定要使用的OpenFlow版本(OFVersion），然后根据每种OpenFlow版本的功能来处理每种情况。

```java
/*
 * We don't know what version the factory is, 
 * but we can detect and account for each scenario.
 * Maybe you want to take advantage of the extended
 * feature-set of OF1.3 if you can, or fall-back to
 * your OF1.0-compatible algorithm if you cannot.
 */
OFVersion detectedVersion = myFactory.getVersion();
switch (detectedVersion) {
    case OF_10:
        /* Begin 1.0 algorithm */
        break;
    ...
    ...
    ...
    case OF_13:
        /* Begin 1.3 algorithm */
        break;
    default:
        /* 
         * Perhaps report an error if
         * you don't want to support
         * a specific OFVersion.
         */
        ...
        break;
}
```

## Matches

匹配在OpenFlow中用于描述或定义数据包头字段的特征。它们在构成FlowMod时最常用，而OpenFlowJ-Loxigen利用构建器模式使构建Matchs变得相当简单。

如上面工厂中所述，几乎所有OpenFlow概念都可以从OFFactory访问。因此，构造Match的先决条件是首先获取对OFFactory的引用，我们将假定您已经拥有该引用。以下是如何构造匹配的示例：

```java
Match myMatch = myFactory.buildMatch()
    .setExact(MatchField.IN_PORT, OFPort.of(1))
    .setExact(MatchField.ETH_TYPE, EthType.IPv4)
    .setMasked(MatchField.IPV4_SRC, IPv4AddressWithMask.of("192.168.0.1/24"))
    .setExact(MatchField.IP_PROTO, IpProtocol.TCP)
    .setExact(MatchField.TCP_DST, TransportPort.of(80))
    .build();
```

如您所见，`OFFactory myFactory`包含可用于构造`Match`的构建器(`Match.Builder`)。您可以分别使用`setExact()`或`setMasked()`来指定要匹配的精确字段或被屏蔽的字段。`setExact()`或`setMasked()`的第一个参数是`MatchField`，它指定要匹配的标头字段。`MatchField`包含可以匹配的各种`OFValueType`。第二个参数是您要匹配的特定标题字段的内容。有关如何使用`OFPort`，`EthType`，`IPv4AddressWithMask`和其他`OFValueType`的信息，请参见下面的值类型。

就像使用`ovs-ofctl`或使用静态流推程序手动编写`FlowMod`时一样，您必须为所使用的每个`MatchField`指定所有必备项。例如，在上面的代码中，必须匹配`EthType.IPv4`才能匹配`IPv4Address`。(在没有第一个`IPv4`数据包的情况下，不可能具有源`IPv4Address`或目标`IPv4Address`的标头字段。)幸运的是，`OpenFlowJ-Loxigen`使先决条件检查过程变得容易。您可以提供一个完全匹配且带有掩码的`MatchField`的`Match`对象，您将其如上所述设置为`arePrerequisitesOK()`函数。每个`MatchField OFValueType`可以在`Match`对象上调用`arePrerequisitesOK()`函数，该函数将检查并验证是否指定了`MatchField`类型的所有先决条件。例如，

`MatchField.IPV4_SRC.arePrerequisitesOK(myMatch);`

如果`Match`对象中包含`IPv4`的以太类型的匹配，则返回`true`。否则，如果在`ethertype = IPv4`上不存在先决条件匹配项，则它将返回`false`。(在上面创建`myMatch Match`对象的示例代码中，我们指定了`ethertype = IPv4`的匹配项，因此在这种情况下，先决条件检查将返回`true`。)

`OpenFlowJ-Loxigen`的一个不错的功能是您指定的`MatchField`的`OFValueType`会强制检查第二个参数，以确保它具有相同的类型。例如，上面的代码中匹配的第一个字段是：

`.setExact(MatchField.IN_PORT, OFPort.of(1))`

`MatchField.IN_PORT`的`OFValueType为OFPort`。如果您还没有为第二个参数指定`OFPort`的`OFValueType`，则Java编译器将报告错误。如果使用的是诸如Eclipse之类的IDE，则应该立即看到编译错误。

Match的构建器的另一个有用功能是，您可以检查`OFFactory`的`OpenFlow`版本是否支持完全匹配或掩码匹配。这可以通过分别调用构建器的`supports()`和`supportsMasked()`函数来完成。如果两者均为假，则`OFFactory`的`OFVersion`(以及交换机所使用的OpenFlow协议版本)根本不支持头字段的匹配。

```java
Match myMatch = myFactory.buildMatch()
    ...
    ...
    .supports(MatchField.IPv6_SRC)
    .supportsMasked(MatchField.ETH_SRC)
    .build();
```

还值得注意的是，`Match`对象和所有`OpenFlowJ-Loxigen`构建的对象的复制非常简单。每个内置对象(例如`Match`)都包含一个名为`createBuilder()`的函数。这将创建一个对象类型的新构建器，该构建器预先设置了从中调用构建器的对象的所有属性。考虑一个例子：

```java
Match myMatch = myFactory.buildMatch()
    .setExact(MatchField.IN_PORT, OFPort.of(1))
    .setExact(MatchField.ETH_TYPE, EthType.IPv4)
    .setMasked(MatchField.IPV4_SRC, IPv4AddressWithMask.of("192.168.0.1/24"))
    .setExact(MatchField.IP_PROTO, IpProtocol.TCP)
    .setExact(MatchField.TCP_DST, TransportPort.of(80))
    .build();
 
Match anotherMatch = myMatch.createBuilder().build();
if ( anotherMatch.equals(myMatch) ) { ... } /* true */
if ( anotherMatch == myMatch ) { ... } /* false */
```

注意，另一个匹配是通过`myMatch`创建的。该构建器是从`myMatch`对象创建的，并且此构建器包含在`myMatch`中设置的所有预设属性/字段。然后立即构建此构建器，从而生成`anotherMatch Match`对象。`anotherMatch`和`myMatch`完全相等；但是，它们是单独的对象。可以用这种方式复制所有`OpenFlowJ-Loxigen`构建的对象。

## 动作

像`Matches`一样，`OFAction`也由`OFFactory`公开。注意：有一个称为`OFAction`的类，另一个名为`OFActions`(带有s)。为了表示`OFAction`的复数形式，我将使用所有格`OFAction`，这在语法上不是正确的，但是很好。在本文档中，所有类似情况都将使用该约定。

`OFActions allActions = myFactory.actions();`

上面对`action()`的此调用返回`OFActions`接口的实现。返回给您的实现完全取决于`OFFactory`的`OFVersion`。每个`OpenFlow`版本都有一个`OFActions`接口的单独实现，该接口特定于该`OpenFlow`版本中支持的操作。话虽如此，因为公开了接口，所以在每个`OFFactory`版本中公开了所有`OpenFlow`协议版本中所有可能的`OFAction`。因此，您应注意仅使用交换机的`OpenFlow`版本支持的`OFAction`。如果您尝试使用交换机的`OpenFlow`协议版本不支持的`OFAction`（可以通过检查`OFVersion`来检测），当您尝试获取无效的`OFAction`的构建器或构造无效的`OFAction`时，`OpenFlowJ-Loxigen`库将为您提供`UnsupportedOperationException`。考虑以下： 

`OFMeterMod.Builder meterModBldr = myFactory.actions().buildMeterMod();`

如果您的`OFFactory myFactory`是`OFVersion.OF_10`(表示您的交换机使用的是`OpenFlow 1.0`)，并且您尝试构建`OFMeterMod`消息，则会抛出`UnsupportedOperationExecption`，因为在`OpenFlow 1.0`中不存在计量器。

`"OFMeterMod not supported in version 1.0"`

现在，让我们讨论有关如何使用`OFAction`的细节。正如上面所暗示的，大多数`OFAction`都有一些构建器，所有构建器都作为`OFActions`接口中的函数公开。也可以不使用构建器而获得`OFAction`。实际上，某些不需要任何应用程序数据作为输入的`OFAction`甚至没有构建器(例如，剥离VLAN)。

## OpenFlow 1.0

让我们考虑一个OpenFlow 1.0示例：

```java
ArrayList<OFAction> actionList = new ArrayList<OFAction>();
OFActions actions = myOF10Factory.actions();
 
/* Use builder to create OFAction. */
OFActionSetDlDst setDlDst = actions.buildSetDlDst()
    .setDlAddr(MacAddress.of("ff:ff:ff:ff:ff:ff")
    .build();
actionList.add(setDlDst);
/* Create OFAction directly w/o use of builder. */
OFActionSetNwDst setNwDst = actions.setNwDst(IPv4Address.of("255.255.255.255"));
actionList.add(setNwDst);
 
/* OFActionStripVlan requires no data, thus is does not have a builder at all. */
OFActionStripVlan stripVlan = actions.stripVlan();
actionList.add(stripVlan);
 
/* Use builder again. */
OFActionOutput output = actions.buildOutput()
    .setMaxLen(0xFFffFFff)
    .setPort(OFPort.of(1))
    .build();
actionList.add(output);
```

上面的示例创建`OFAction`的列表。需要从应用程序获取数据的`OFAction`可以在使用或不使用构建器的情况下进行组合；为了方便起见，提供了这两种方法，并且可以互换使用。像`OFActionStripVlan`这样的`OFAction`根本不接受任何数据作为输入，根本没有构建器。请注意，所有特定`OFAction`的扩展`OFAction`，这就是将它们存储在`OFAction`类型的单个`ArrayList`中的方式。正如您将在后续主题中看到的那样，您可以将此`OFAction`列表提供给其他类型(例如`OFFlowMods`)。

## OpenFlow 1.3

`OpenFlow 1.2`引入了`OXM`或`OpenFlow`可扩展匹配。对于`OXM`，所有导致修改现有标头字段的操作都由新的`set-field`操作指定，其中`set-field`操作由指定标头字段的`OXM`和要写入该标头字段的新值组成。为此，`OpenFlowJ-Loxigen`具有`OFAction OFActionSetField`。`OFActionSetField`采用一个`OFOxm`来定义新值和写入新值的字段。与`OFAction`一样，可以从`OFFactory`访问`OFOxm`，但可以通过`oxms()`函数访问。该调用返回由特定于`OFactory`的`OFVersion`的实现支持的`Oxms`接口。请注意，`Oxm`和`Oxms`之间的区别类似于`OFAction's`和`OFActions`。`Oxms`是包含所有单个`Oxm`的界面。 让我们完成上面与`OpenFlow 1.3`相同的`OpenFlow 1.0`示例。

```java
ArrayList<OFAction> actionList = new ArrayList<OFAction>();
OFActions actions = myOF13Factory.actions();
OFOxms oxms = myOF13Factory.oxms();
 
/* Use OXM to modify data layer dest field. */
OFActionSetField setDlDst = actions.buildSetField()
    .setField(
        oxms.buildEthDst()
        .setValue(MacAddress.of("ff:ff:ff:ff:ff:ff"))
        .build()
    )
    .build();
actionList.add(setDlDst);
 
/* Use OXM to modify network layer dest field. */
OFActionSetField setNwDst = actions.buildSetField()
    .setField(
        oxms.buildIpv4Dst()
        .setValue(IPv4Address.of("255.255.255.255"))
        .build()
    )
    .build();
actionList.add(setNwDst);
 
/* Popping the VLAN tag is not an OXM but an OFAction. */
OFActionPopVlan popVlan = actions.popVlan();
actionList.add(popVlan);
 
/* Output to a port is also an OFAction, not an OXM. */
OFActionOutput output = actions.buildOutput()
    .setMaxLen(0xFFffFFff)
    .setPort(OFPort.of(1))
    .build();
actionList.add(output);
```

比较这两个示例，您可以看到在`OpenFlow 1.3`中，`OFOxm`与`OFActionSetFields`一起用于指定目标MAC和IP地址的重写，因为它们是字段修改。另一方面，删除VLAN标签并将数据包发送到交换机端口的定义不是`OFOxm`，而是`OFAction`。

## 使用说明

`OpenFlow 1.1`的出现引入了`OFInstruction`的功能，在`OpenFlow 1.3`中对其进行了稍微扩展。OF指令可以是以下任意一种：

- OFInstructionApplyActions：根据提供的`OFAction`列表立即修改数据包
- OFInstructionWriteActions：将动作列表与数据包相关联，但等待执行它们
- OFInstructionClearActions：清除与数据包关联的待处理操作列表
- OFInstructionGotoTable：将数据包发送到特定流表
- OFInstructionWriteMetadata：随着数据包在处理管道中的前进，在数据包中保存一些元数据
- OFInstructionExperimenter：允许扩展`OFInstruction`集
- OFInstructionMeter：将数据包发送到仪表
- OFInstruction是`OFAction`之上的一层。用于`OpenFlow 1.1+`的`OFFlowMod`由`OFInstruction`的列表组成，而没有`OFAction`的明确列表。在`OFInstructionApplyActions`或`OFInstructionWriteActions`中指定要应用于与流匹配的数据包的任何`OFAction`。最常见的情况是`OFInstructionApplyActions`，它会根据`OFAction`的列表立即修改数据包。

所有`OFInstruction`都可以通过`notes()`函数从`OFFactory`访问。该函数调用返回一个`OFInstructions`接口。此接口的实现将特定于`OFFactory`的`OpenFlow`版本。

`OFInstructions instructions = myFactory.instructions();`

像`OFActions`接口实现一样，在尝试构造对该特定`OFVersion`不支持的任何`OFInstruction`的情况下，`OFInstructions`将引发`UnsupportedOperationException`。例如，如果`myFactory`属于`OFVersion.OF_10`，则任何尝试创建特定指令的尝试都将导致`UnsupportedOperationException`。

`myOF10Factory.instructions().buildApplyActions();`

结果是：

`"OFInstructionApplyActions not supported in version 1.0"`

如果为`OFVersion`小于`OFVersion.OF_13`的`OFFactory`调用`buildMeter()`，则会发生相同的情况。直到`OpenFlow 1.3`才引入计量器。

如上所述，`OFAction`的列表可以包含在`OFInstructionApplyActions`或`OFInstructionWriteActions`对象中。考虑以下基于`Actions`中的`OpenFlow 1.3`示例的示例：

```java
OFInstructions instructions = myOF13Factory.instructions();
ArrayList<OFAction> actionList = new ArrayList<OFAction>();
OFActions actions = myOF13Factory.actions();
OFOxms oxms = myOF13Factory.oxms();
 
/* Use OXM to modify data layer dest field. */
OFActionSetField setDlDst = actions.buildSetField()
    .setField(
        oxms.buildEthDst()
        .setValue(MacAddress.of("ff:ff:ff:ff:ff:ff"))
        .build()
    )
    .build();
actionList.add(setDlDst);
 
/* Use OXM to modify network layer dest field. */
OFActionSetField setNwDst = actions.buildSetField()
    .setField(
        oxms.buildIpv4Dst()
        .setValue(IPv4Address.of("255.255.255.255"))
        .build()
    )
    .build();
actionList.add(setNwDst);
 
/* Popping the VLAN tag is not an OXM but an OFAction. */
OFActionPopVlan popVlan = actions.popVlan();
actionList.add(popVlan);
 
/* Output to a port is also an OFAction, not an OXM. */
OFActionOutput output = actions.buildOutput()
    .setMaxLen(0xFFffFFff)
    .setPort(OFPort.of(1))
    .build();
actionList.add(output);
 
/* Supply the OFAction list to the OFInstructionApplyActions. */
OFInstructionApplyActions applyActions = instructions.buildApplyActions()
    .setActions(actionList)
    .build();
```

## FlowMods

`OpenFlowJ-Loxigen`库公开了版本无关的`OFFlowMod`接口，该接口具有多个子接口，这些子接口允许您编写与特定`OFFlowModCommand`对应的特定类型的`OFFlowMod`：

- OFFlowAdd
- OFFlowModify
- OFFlowModifyStrict
- OFFlowDelete
- OFFlowDeleteStrict

像我们已经讨论过的其他`OpenFlow`概念和类型一样，`OFFlowMod`接口是针对每个`OpenFlow`版本单独实现的。以这种方式，您的`OFFlowMod`的`OpenFlow`版本不支持的任何`OFFlowMod`操作都将引发`UnsupportedOperationException`。(例如，在`OFVersion.OF_10`的`OFFlowAdd`中设置`OFInstruction`列表将引发`UnsupportedOperationException`。直到`OpenFlow 1.1`才引入`OFInstruction`。)

关于`OFFlowMods`的重要说明：`OpenFlow 1.3`消除了将数据包转发到控制器的默认表丢失行为。取而代之的是，`OpenFlow 1.3`指定控制器应插入优先级为零的特殊表丢失流，所有`MatchFields`通配，并且由任何`OFAction`定义动作列表。为了支持在控制器中处理不匹配的表丢失数据包，`Floodlight`将插入优先级为零，未设置`MatchFields`的流以及单个输出`OFAction`，以将数据包发送到控制器。在表中所有其他流与数据包都不匹配的情况下，该数据流将用作"包罗万象"(`catch-all`)流，并将数据包作为数据包输入消息发送给控制器。与`OpenFlow 1.0-1.2`（此行为嵌入在流表本身中）不同，在`OpenFlow 1.3`中，显然是使用实际流来实现的，所有模型都可以看到和修改。因此，编写`OpenFlow 1.3`的`OFFlowMods`时应格外小心。您应该确保它们不会替换或删除默认的表缺失流。否则，控制器以及其他模块可能无法从交换机接收任何入站消息。

关于"`Strict`"的附带说明：根据`OpenFlow`规范，流修改消息和流删除消息支持"`Strict`"版本和"`No-Strict`"版本。"`Strict`"是指与您指定的匹配完全相似的任何流，然后将修改或删除该流。如果您选择"`No-Strict`"版本，则意味着任何看起来像您指定的匹配项的流（即，该流至少包含您指定的匹配项，但可能包含更多匹配项）将被修改或删除。 

此外，以与前面介绍的`OpenFlow`概念和类型类似的方式，`OFFactory`可用于生成每种类型的`OFFlowMod`。

`OFFlowAdd flowAdd = myFactory.buildFlowAdd()`

`OFFactory`将为您指定的`OFFlowMod`返回一个生成器。在某些情况下，您可能会发现将`OFFlodMod`从一种`OFFlowModCommand`类型转换为另一种类型很有用。目前在`OpenFlowJ-Loxigen`中这是不可能的。但是，`net.floodlightcontroller.util.FlowModUtils.java`中的`FlowModUtils`类提供了几个辅助函数，这些函数可将任何OFFlowMod转换为上面列出的任何特定`OFFlowMod`：

```java
OFFlowAdd flowAdd = myFactory.buildFlowAdd().build();
 
/* Convert from any extension of OFFlowMod to another. */
OFFlowModify flowModify = FlowModUtils.toFlowModify(flowAdd);
OFFlowModifyStrict flowModifyStrict = FlowModUtils.toFlowModifyStrict(flowAdd);
OFFlowDelete flowDelete = FlowModUtils.toFlowDelete(flowAdd);
OFFlowDeleteStrict flowDelStrict = FlowModUtils.toFlowDeleteStrict(flowAdd);
OFFlowAdd flowAdd2 = FlowModUtils.toFlowAdd(flowModify);
```

现在，让我们简要地讨论如何组成`OFFlowMod`。可以按照与我们讨论的所有其他`OFObject`相同的方式来组成/构建所有`OFFlowMod`。`OFFactory`返回的构建器包含许多`setter`函数，用于设置`OFFlowMod`的所有可能字段。以这个`OpenFlow 1.3`为例：

```java
OFFlowAdd flowAdd = my13Factory.buildFlowAdd()
    .setBufferId(OFBufferId.NO_BUFFER)
    .setHardTimeout(3600)
    .setIdleTimeout(10)
    .setPriority(32768)
    .setMatch(myMatch)
    .setInstructions(myInstructionList)
    .setTableId(TableId.of(1))
    .build();
```

还有其他设置器可以用于执行操作，例如指定输出`OFGroup`，标志等。

组成`OFFlowMod`后，您可能需要将其发送到交换机。所有`OFFlowMods`都是`OFMessages`，因此，它们可以直接写入开关：

```java
/* Continuing with the previous OFFlowAdd example... */
...
IOFSwitch mySwitch = switchService.getSwitch(DatapathId.of(1));
mySwitch.write(flowAdd);
```

如果要插入类似的OFFlowMods，该怎么办？如果这样做，则无需从头开始编写它们。只需从现有的OFFlowMod创建另一个构建器，更改所需的属性，然后构建新对象。

```java
OFFlowAdd flowAdd = my13Factory.buildFlowAdd()
    .setBufferId(OFBufferId.NO_BUFFER)
    .setHardTimeout(3600)
    .setIdleTimeout(10)
    .setPriority(32768)
    .setMatch(myMatch)
    .setInstructions(myInstructionList)
    .setTableId(TableId.of(3))
    .build();
 
/* Create copy and modify slightly. */
OFFlowAdd flowAdd2 = flowAdd.createBuilder()
    .setInstructions(newInstructionList)
    .build();
```

在上面的示例中，在编写`flowAdd2`时，`flowAdd`用作起点。除了在`flowAdd2`的构建器中显式设置的属性外，`flowAdd`的所有属性都将保留。除了`OFInstruction`列表和输出端口外，两个`OFFlowAdds`的所有属性都相同。

最后一个值得注意的重要事情是：尽管`OpenFlow 1.1`版和更高版本在`OFInstructionApplyActions`和`OFInstructionWriteActions OFInstructions`中包装了`OFAction`，但`OpenFlowJ-Loxigen`确实允许您直接为`OFFlowMod`指定动作列表。该库将自动在`OFInstructions`列表中插入作为`OFInstructionApplyActions`提供的`OFAction`的任何列表。因此，以下两个实现产生了等效的`OFFlowAdd`对象。

(1)允许`OpenFlowJ-Loxigen`为您创建`OFInstructionApplyActions`并将其插入`OFInstruction`列表：

```java
/*
 * Upon build, the builder will automatically insert the
 * supplied OFAction list as an OFInstructionApplyActions
 * in the list of OFInstructions.
 */
OFFlowAdd flowAdd13 = myFactory.buildFlowAdd()
    ...
    .setActions(myActionList)
    .build();
```

(2)在`OFInstruction`列表中明确提供`OFInstructionApplyAction`：

```java
OFInstructionApplyActions applyActions = my13Factory.instructions().buildApplyActions()
    .setActions(myActionList)
    .build();
ArrayList<OFInstruction> instructionList = new ArrayList<OFInstruction>();
instructionList.add(applyActions);
OFFlowAdd flowAdd13 = myFactory.buildFlowAdd()
    ...
    .setInstructions(instructionList)
    .build();
```

实现(1)可能更难解密，因为尚不清楚为您的代码编写的`OpenFlow`版本。另一方面，实现(2)比较麻烦并且需要更多代码。使用`OpenFlow 1.1`或更高版本时需要选择，并且需要立即应用`OFAction`。请注意，此快捷方式仅适用于`OFInstructionApplyActions`，它会将`OFAction`立即应用于数据包。必须在`OFInstruction`的列表中显式提供`OFInstructionWriteAction`和所有其他操作。

## Groups

`OFGroups`最初是在`OpenFlow 1.1`中引入的，在`OpenFlow 1.3`中进行了较小的扩展。`OFGroup`旨在简化`OpenFlow`交换机中并使之变得更复杂的操作，例如复制数据包，将不同的`OFAction`集应用于单个数据包，负载平衡以及检测和处理链路故障。为此，将`OFGroup`构造为`OFAction`列表的列表，或者将其构造为`OFAction`的2D列表。外部列表称为存储桶列表，它由`OFBuckets`组成。存储桶列表中的每个`OFBucket`都包含`OFAction`的列表。所分配的`OFGroup`的类型将决定`OFBucket`的使用方式。`OFGroup`有四种不同类型：

1. OFGroupType.ALL：向每个`OFBucket`提供数据包的副本，并将每个`OFBucket`中的`OFAction`列表应用于数据包的`OFBucket`。
2. OFGroupType.SELECT：使用开关确定（通常为轮询）方法对所有`OFBucket`之间的数据包进行负载平衡。可以为分组的加权循环分配分配权重。
3. OFGroupType.INDIRECT：只允许一个`OFBucket`，并且应用所有`OFAction`。当许多流包含相同的操作集时，这可以实现更有效的转发。与单个`OFBucket`的全部相同。
4. OFGroupType.FF：快速故障转移。使用单个`OFBucket`，如果活动`OFBucket`的特定链接或指定`OFGroup`中的链接失败，则自动切换到`OFGroup`中的下一个`OFBucket`。 

为了在`OpenFlow`交换机上使用`OFGroup`，您需要为交换机配置所需的`OFGroup`数量和类型。所有`OFGroup`消息都可以从与交换机正在使用的`OpenFlow`协议版本相对应的`OFFactory`访问。像我们已经讨论过的其他类型一样，如果您的`OFFactory`的`OFVersion`不支持`OFGroups`或您希望执行的操作，则会引发`UnsupportedOperationException`。

与`OFFlowMod`一样，`OpenFlowJ-Loxigen`公开了版本无关的`OFGroupMod`，它具有多个子接口，可让您编写与特定`OFGroupModCommand`对应的特定类型的`OFGroupMod`：

1. OFGroupAdd
2. OFGroupModify
3. OFGroupDelete

这些`OFGroupMods`是`OFMessage`，可以像`OFFlowMods`一样编写并写入到交换机中。

```java
ArrayList<OFGroupMod> groupMods = new ArrayList<OFGroupMod>();
 
OFGroupAdd addGroup = myFactory.buildGroupAdd()
    .setGroup(OFGroup.of(1))
    .setGroupType(OFGroupType.ALL)
    .build();
 
groupMods.add(addGroup);
 
OFGroupModify modifyGroup = myFactory.buildGroupModify()
    .setGroup(OFGroup.of(2))
    .setGroupType(OFGroupType.SELECT)
    .build();
groupMods.add(modifyGroup);
 
OFGroupDelete deleteGroup = myFactory.buildGroupDelete()
    .setGroup(OFGroup.of(3))
    .setGroupType(OFGroupType.INDIRECT)
    .build();
groupMods.add(deleteGroup);
 
IOFSwitch mySwitch = switchService.getSwitch(DatapathId.of(1));
mySwitch.write(groupMods);
```

添加`OFGroup`时，您还可以指定要包含在`OFGroup`中的`OFBucket`列表。根据定义，`OFGroupMod`将用指定的`OFBucket`列表替换整个`OFGroup`的`OFBucket`列表。向`OFGroupDelete`提供`OFBucket`列表没有任何意义。整个`OFGroup`将按照`OpenFlow`规范中的定义删除。当然，如果将`OFBuckets`列表提供给`OFGroupAdd OFMessage`，则将其包含在新`OFGroup`中。

```java
ArrayList<OFBucket> bucketList = new ArrayList<OFBucket>();
ArrayList<OFBucket> singleBucket = new ArrayList<OFBucket>();
...
...
/* Assume for the time being the bucketList has been magically populated with OFBuckets. */
OFGroupAdd addGroup = myFactory.buildGroupAdd()
    .setGroup(OFGroup.of(1))
    .setGroupType(OFGroupType.ALL)  .setBuckets(bucketList) /* Will be included with the new OFGroup. */
    .build();
 
OFGroupModify modifyGroup = myFactory.buildGroupModify()
    .setGroup(OFGroup.of(2))
    .setGroupType(OFGroupType.SELECT)
    .setBuckets(bucketList) /* Will replace the OFBucket list defined in the existing OFGroup. */
    .build();
 
OFGroupModify modifyGroup2 = myFactory.buildGroupModify()
    .setGroup(OFGroup.of(3))
    .setGroupType(OFGroupType.INDIRECT)
    .setBuckets(singleBucket) /* INDIRECT only supports a single OFBucket; will replace an OFBucket or assign one if not present in the OFGroup already. */
    .build();
 
OFGroupAdd addGroup2 = myFactory.buildGroupAdd()
    .setGroup(OFGroup.of(4))
    .setGroupType(OFGroupType.FF)
    .setBuckets(bucketList) /* Will be included with the new OFGroup. */
    .build();
```

关于`OFGroups`的棘手部分是，您需要牢记`OFGroupType`来指定每个`OFBucket`的属性。每个`OFGroupType`的行为都不同，因此需要适当设置`OFBucket`属性。像我们讨论过的所有其他类型一样，`OFBuckets`可以从`OFFactory`中构建。

```java
OFBucket myBucket = myFactory.buildBucket()
    .setActions(myActionList)
    .setWatchGroup(OFGroup.of(1))
    .setWatchPort(OFPort.of(48))
    .setWeight(100)
    .build();
```

如上所示，`OFBuckets`包含`OFAction`的列表，要监视的`OFGroup`，要监视`的OFPort`和权重。以下是有关如何为`OFGroupType.ALL`和`OFGroupType.INDIRECT`组成`OFBucket`的示例：

```java
OFBucket myBucket = myFactory.buildBucket()
    .setActions(myActionList)
    .setWatchGroup(OFGroup.ANY) /* ANY --> don't care / wildcarded. */
    .setWatchPort(OFPort.ANY) /* ANY --> don't care / wildcarded. */
    .build();
```

尽管未为`OFGroupType.ALL`定义监视`OFGroups`和`OFPorts`，但是`OpenFlowJ-Loxigen`要求您明确指定`ANY OFGroup和OFPort`类型。指定重量没有定义，也不是必需的。

现在，让我们考虑一下`OFBucket`是如何构成`OFGroupType.SELECT`的`OFGroup`的：

```java
OFBucket myBucket = myFactory.buildBucket()
    .setActions(myActionList)
    .setWatchGroup(OFGroup.ANY) /* ANY --> don't care / wildcarded. */
    .setWatchPort(OFPort.ANY) /* ANY --> don't care / wildcarded. */
    .setWeight(89) /* Relative weight used to govern packet distribution to OFBuckets in the OFGroup. */
    .build();
```

同样，手表`OFGroup`和`OFPort`通过`ANY`通配；但是，这次我们为`OFBucket`分配权重，以告知交换机的`OFBucket`选择机制该特定`OFBucket`的相对优先级。如果为所有`OFBucket`分配了相等的权重，则`OFGroup`的`OFBucket`列表中所有`OFBucket`之间的数据包分布是相等的。交换机将进入`OFGroup`的数据包分发到`OFBucket`的机制不在`OpenFlow`协议的范围内，并且取决于特定的交换机实现。在`OpenFlow`规范中对`OFGroupType.SELECT OFGroup`提出/​​推荐了加权轮询方法。

最后，让我们考虑一下`OFGroupType.FF`（快速故障转移）：

```java
OFBucket myBucket = myFactory.buildBucket()
    .setActions(myActionList)
    .setWatchGroup(OFGroup.of(23)) /* Monitor liveness of OFGroup 23. */
    .setWatchPort(OFPort.of(7)) /* Monitor liveness of OFPort 7. */
    .build();
```

`OFGroupType.FF`设计为如果当前`OFBucket`在指定观察端口遇到链接故障和/或如果被观察的`OFGroup`经历链接故障，则允许快速故障转移到下一个`OFBucket`。这样，这个特定的`OFBucket`需要将其手表`OFGroup`和/或手表`OFPort`设置为一些有意义的值。如果只需要手表`OFGroup`或手表`OFPort`中的一个，则应将另一个设置为`OFGroup.ANY`或`OFPort.ANY`。

最后，就像我们讨论过的其他任何类型一样，构建器模式为`OFGroupMods`和`OFBucket`提供了一种简单的复制机制。

```java
OFGroupAdd addGroup = myFactory.buildGroupAdd()
    .setGroupType(OFGroupType.ALL)
    .setGroup(OFGroup.of(50))
    .setBuckets(myBucketList)
    .build();
OFGroupAdd addGroup2 = addGroup.createBuilder() /* Builder contains all fields as set in addGroup. */
    .setGroup(OFGroup.of(51))
    .setBuckets(myOtherBucketList)
    .build(); /* Will create addGroup2, only changing the OFGroup number and the OFBucket list. */
 
OFBucket myBucket = myFactory.buildBucket()
    .setActions(myActionList)
    .setWatchGroup(OFGroup.ANY)
    .setWatchPort(OFPort.ANY)
    .build();
OFBucket myBucket2 = myBucket.createBuilder() /* Builder contains all fields as set in myBucket. */
    .setActions(myOtherActionList)
    .build(); /* Will create myBucket2 the same as myBucket, only changing the OFAction list. */
```

## PacketIns

任何模块都可以注册为`IOFMessageListener`。这样，每当从连接的交换机接收到`OFMessage`时，控制器就会通知该模块。最常见的情况是希望对交换机发送的不匹配数据包（也称为`PacketIn`消息）做出反应的模块。`OpenFlowJ-Loxigen`将`PacketIn`消息作为`OFPacketIn`实例呈现给您的侦听模块。`OFPacketIns`就像其他任何`OpenFlow`对象一样，它们是不可变的，并且是由构建器构造的。`OFPacketIn`消息的构造是由控制器为您的模块完成的，因此在此我们不再赘述。(但是，如果您想构建`OFPacketIn`，则可以像获取其他`OpenFlow`概念一样，通过获取`OFFactory`并对其进行构造来实现。)不过，从`OFPacketIn`消息中可以得到一些有用的东西。

```java
/**
  * The controller will invoke the receive()
  * function automatically when an OFMessage is
  * received from a switch if the module
  * implements the IOFMessageListener interface
  * and is registered with the controller as an
  * IOFMessageListener.
  *
Command receive(IOFSwitch sw, OFMessage msg, FloodlightContext cntx) {
    switch (msg.getType()) {
            case PACKET_IN:
                OFPacketIn myPacketIn = (OFPacketIn) msg;
            OFPort myInPort = 
                (myPacketIn.getVersion().compareTo(OFVersion.OF_12) < 0) 
                ? myPacketIn.getInPort() 
                : myPacketIn.getMatch().get(MatchField.IN_PORT);
            ...
            ...
            break;
              default:
                    break;
        }
    return Command.CONTINUE;
}
```

首先请注意，`OFMessage`是由`IOFMessageListener`接口的`receive()`函数传递的。因为`OFPacketIn`扩展了`OFMessage`，所以我们可以通过`msg.getType()`检测到`OFMessage`消息确实是`OFPacketIn`，然后将其转换为`OFPacketIn`。

现在，这是棘手的部分，您可以在控制器中的几乎每个`IOFMessageListener`实现模块中看到它的演示-您如何检测数据包到达交换机的`OFPort`？如果`OpenFlow`协议版本为`OpenFlow 1.0`或`1.1`，则将入口端口定义为`OFPacketIn`消息中的不同字段，可通过`getInPort()`函数进行访问。另一方面，如果`OpenFlow`版本为`1.2`或更高版本，则将入口端口作为`OXM`包含在`OFPacketIn`的`Match`对象中。因此，您需要小心并检查`OpenFlow`版本，以了解获取入口`OFPort`的正确方法。上面的三元运算符是如何在整个控制器中执行检查的，这是一种很好且紧凑的方式。

## PacketOuts

从控制器发送数据包通常是有益的，该数据包将被注入到特定交换机的数据平面中。`OFPacketOut`是如何完成此任务的方法。像所有`OpenFlow`概念一样，可以从`OFFactory`构造`OFPacketOut`。

根据定义，`OFPacketOut是OFMessage`，并且应该由交换机从控制器接收，这时交换机将获取`OFPacketOut`的有效负载并将其发送到`OFPacketOut`指定的任何端口。因此，`OFPacketOut`应该包含一些表示有效数据包的数据。以下是用于构造和使用`OFPacketOut`消息的典型工作流程。

```java
include net.floodlightcontroller.packet.*; /* All network packets are defined in the packet package. */
...
...
 
/* Compose L2 packet. */
Ethernet eth = new Ethernet();
eth.setSourceMACAddress(MacAddress.of(1));
eth.setDestinationMACAddress(MacAddress.of(2));
...
...
 
/* Compose L3 packet. */
IPv4 ipv4 = new IPv4();
ipv4.setSourceAddress(IPv4Address.of("192.168.1.1"));
ipv4.setDestinationAddress(IPv4Address.of(0xffFFffFF));
...
...
 
/* Set L2's payload as the L3 packet. */
eth.setPayload(ipv4);
...
...
 
/* Specify the switch port(s) which the packet should be sent out. */
OFActionOutput output = myFactory.actions().buildOutput()
    .setPort(OFPort.FLOOD)
    .build();
 
/* 
 * Compose the OFPacketOut with the above Ethernet packet as the 
 * payload/data, and the specified output port(s) as actions.
 */
OFPacketOut myPacketOut = factory.buildPacketOut()
    .setData(eth.serialize())
    .setBufferId(OFBufferId.NO_BUFFER)
   .setActions(Collections.singletonList((OFAction) output))
    .build();
...
...
 
/* Write the packet to the switch via an IOFSwitch instance. */
mySwitch.write(myPacketOut);
```

如上所示，`net.floodlightcontroller.packet`包包含许多有用的类，这些类用于抽象化从传输层到数据/链接层的数据包构成细节。在这里，我们创建一个以太网实例，并在其中放置一个`IPv4`实例作为其有效负载。然后，我们从`OFFactory`构造`OFPacketOut`对象，并将其有效负载设置为以太网数据包（其中包含`IPv4`数据包）。调用`eth.serialize()`时，以太网数据包及其有效负载会序列化为`byte []`，这是`OFPacketOut`的数据。这些操作指定将发送以太网数据包的端口（在这种情况下为`FLOOD`）。当交换机收到`OFPacketOut OFMessage`消息时，它将从`OFPacketOut`中删除数据并执行其`OFAction`列表中的所有`OFAction`。

## 值类型

在`OpenFlowJ-Loxigen`中，值类型是在`OpenFlow`版本之间保持不变的类型。这些包括但不限于IP地址，MAC地址，交换机端口，数据路径ID等。所有值类型都是不可变的，但它们与上述其他类型的区别在于它们不使用构建器来构造它们。而是显示它们是由`SomeValueType.of()`机制创建的。此外，值类型在大多数情况下是独立的，并且包括用于处理特定值类型的大多数所有实用程序。例如，`IPv4Addresses`包括执行按位操作，子网掩码，CIDR，与`IPv4Address`类型转换为String，整数和字节数组的功能，列表继续。

以下是一些有关如何使用选择值类型的有限 示例。对于所有可能的值类型，请查看`org.projectfloodlight.openflow.types`，其中包含`OpenFlowJ-Loxigen`中的所有值类型。请参阅`OpenFlowJ-Loxigen`文档以获取每种值类型的完整功能集。尽管下面可能没有明确说明，但是大多数值类型都可以从String，十六进制和数字（long，int等）形式创建并转换为String（十六进制）和数字（long，int等）形式，这对于特定的Value Type来说是有意义的。

## 数据路径ID

```java
DatapathId dpid1 = DatapathId.of(1); /* Create a DatapathId from a long. */
DatapathId dpid2 = DatapathId.of("00:00:00:00:00:00:00:02"); /* ...from a String. */
DatapathId dpid3 = DatapathId.of(new byte[] {0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x03}); /* ...from a byte array. */
 
String dpid1String = dpid1.toString(); /* Return the DPID in proper String format, "00:00:00:00:00:00:00:01". */
long dpid2Long = dpid2.getLong(); /* Return the DPID as a long, 2. */U64 dpid3ULong = dpid3.getUnsignedLong(); /* Return the DPID as an unsigned long, 3. */
byte[] dpid3Bytes = dpid3.getBytes(); /* Return the DPID as a byte array, {0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x03}. */
```

## IPv4Address（也用于IPv6Address）

```java
IPv4Address ip1 = IPv4Address.of(1); /* ...from integer. */
IPv4Address ip2 = IPv4Address.of("0.0.0.2"); /* ...from String. */
IPv4Address mask = IPv4Address.of(new byte[] {0xFF, 0xFF, 0xFF, 0x00}); /* ...from byte array. */
 
IPv4Address ipMasked = ip1.applyMask(mask); /* Get IP with mask. */
IPv4Address ipFromCIDR = IPv4Address.ofCidrMaskLength(24); /* Same as "mask" above. */
if ( mask.isBroadcast() ) { ... } /* Is the IP 255.255.255.255? (no) */
 
if ( mask.equals(ipFromCIDR) ) { ... } /* Deep equality check. */
IPVersion version = ip2.getVersion(); /* Returns enum IPVersion.IPv4. */
IPv4Address ip3 = ip1.or(ip2); /* Bitwise OR: 0.0.0.1 | 0.0.0.2 --> 0.0.0.3. */
```

## IPv4AddressWithMask (also for IPv6AddressWithMask)

```java
/* Continuing from IPv4Address above... */
 
IPv4AddressWithMask ipAndMask = IPv4AddressWithMask.of(IPv4Address.of("192.168.0.1"), mask); /* Contains 192.168.0.1 w/mask 255.255.255.0. */
IPv4AddressWithMask ipAndMask2 = IPv4AddressWithMask.of("10.0.0.1/16"); /* Accepts CIDR notation for mask. */
if ( ipAndMask.contains(ip2) ) { ... } /* Check if ip2 is in ipAndMask's subnet. (false) */
```
## Mac地址

```java
MacAddress mac1 = MacAddress.of("FF:ff:FF:ff:FF:ff"); /* ...from String. */
MacAddress mac2 = (MacAddress.of("11:22:33:44:55:66")).applyMask(mac1); /* Apply a mask. */
String macStr = mac2.toString(); /* Returns String "11:22:33:44:55:66". */
 
if ( mac1.isBroadcast() ) { ... } /* Is the MAC all 1's? (yes) */
if ( mac2.isMulticast() ) { ... } /* Does MAC have a 1 in LSB of 1st octet? XXXXXXX1:XX:XX:XX:XX:XX (yes) */
if ( mac2.isLLDPAddress() ) { ... } /* Does the MAC look like 01:80:C2:00:00:0X, ignoring the LS-Byte? (no) */
```

## OFPort

```java
OFPort port1 = OFPort.of(1); /* Port numbers 1 through 48 are precached for efficiency. */
OFPort port2 = OFPort.ofShort((short) 2);
 
OFPort pLocal = OFPort.LOCAL; /* 66534 */
OFPort pZero = OFPort.ZERO; /* Wildcarded default for OF1.0; not used in OF1.1+. */
OFPort pAny = OFPort.ANY; /* a.k.a. "NONE" in OF1.0; used in wildcarding. */
OFPort pAll = OFPort.ALL; /* All physical ports except input port. */
OFPort pCont = OFPort.CONTROLLER;
OFPort pFlood = OFPort.FLOOD; /* All physical ports in VLAN, except input and those that are down/blocked. */
OFPort pTable = OFPort.TABLE; /* In PacketOuts, send to first flow table. */
OFPort pIn = OFPort.IN_PORT;
OFPort pNorm = OFPort.NORMAL; /* Process as L2/L3 learning switch. */
 
int raw = port1.getPortNumber();
short raw2 = port1.getShortPortNumber();
```

## EthType

```java
EthType et1 = EthType.of(2048); /* Decimal IPv4 ethertype. */
EthType et2 = EthType.of(0x806); /* Hexadecimal ARP ethertype. */
EthType et3 = EthType.IPv4; /* Almost every common ethertype is precached/predefined. */
 
if ( et1.compareTo(et3) == 0 ) { ... } /* et1 is the same ethertype as et3. */if ( et1.equals(et3) ) { ... } /* et1 does equal et3. */
if ( et1 == et3 ) { ... } /* et1 and et3 are the SAME OBJECT, since all common, predefined ethertypes are precached. */
```

## IpProtocol

```java
IpProtocol ipp1 = IpProtocol.of(6); /* Decimal TCP IP protocol. */
IpProtocol ipp2 = IpProtocol.of(0x11); /* Hexadecimal UDP IP protocol. */
IpProtocol ipp3 = IpProtocol.TCP; /* Like EthType, almost every common IP protocol is precached/predefined. */
 
if ( ipp1.compareTo(ipp3) == 0 ) { ... } /* ipp1 is the same IP protocol as ipp3. */
if ( ipp1.equals(ipp3) ) { ... } /* ipp1 does equal ipp2. */
if ( ipp1 == ipp3 ) { ... } /* ipp1 and ipp3 are the SAME OBJECT, since all common, predefined IP protocol numbers are precached. */
```

## TransportPort

```java
TransportPort tp1 = TransportPort.of(80); /* Decimal transport port number; not associated with any specific protocol. */
TransportPort tp2 = TransportPort.of(-1); /* Throws IllegalArgumentException; not possible to get a TransportPort outside MAX_PORT and MIN_PORT. */
TransportPort tp3 = TransportPort.MAX_PORT;
TransportPort tp4 = TransportPort.MIN_PORT;
 
int raw = tp1.getPort(); /* Get integer port value. */
```

其他包括VlanVid，OFGroup，TableId，U8，U16，U32，U64，U128，还有更多……请查看org.projectfloodlight.openflow.types以获取完整列表和API。[此处提供了适用于OpenFlowJ-Loxi的Javadoc](https://floodlight.atlassian.net/wiki/spaces/floodlightcontroller/pages/1343622/Javadoc+Entry)。

