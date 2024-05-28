# 使用可扩展消息和在场协议（XMPP）的隐写术
> **[*Steganography using the Extensible Messaging and Presence Protocol (XMPP)*](https://doi.org/10.48550/arXiv.1310.0524).by [Reshad Patuck](https://arxiv.org/search/cs?searchtype=author&query=Patuck,+R), [Julio Hernandez-Castro](https://arxiv.org/search/cs?searchtype=author&query=Hernandez-Castro,+J)**. *School of Computing, University of Kent, Cornwallis South Building, Canterbury, CT2 7NF, UK*
### 摘要
我们首次提出了在可扩展消息和在场协议（XMPP）中隐藏数据的不同机制。这是一种非常流行的即时通讯协议，被 Google Talk、Cisco、LiveJournal 等许多通讯平台所使用。本文介绍了如何在不引起任何中间人怀疑的情况下，从一个 XMPP 客户端向另一个客户端发送秘密信息。与其他试图在即时信息内容中隐藏数据的相关工作不同，本文描述的方法主要侧重于使用底层协议作为隐写术的手段。通过这种方法，我们提供了一种更强大的数据隐藏手段，并对其一般安全性进行了初步分析，特别是针对基于熵的隐写分析。

**关键词**：隐写术、隐分析、隐蔽信道、数据隐藏、XMPP、即时信息、协议
## 1 简介
### 1.1 隐写术
隐写术（Steganography）是一门交换隐藏消息的科学和艺术，这种交换方式除了发送者和预期接收者之外，没有人会怀疑任何隐藏数据的存在。Steganography 一词源于希腊语，*Steganos* 意为 "覆盖或保护"，*Graphei* 意为 "书写"。

与密码学不同，隐写术只涉及隐藏信息本身的存在。隐写术的目的不是使数据无法读取，而是完全隐藏这些数据。理想的隐写信息看起来与普通的干净信息完全相同，不会引起任何怀疑[^1]。
#### 1.1.1 历史
希腊历史学家希罗多德（Herodotus）向我们讲述了爱奥尼亚（Ionia）的起义（约公元前 499 年，ca. 499 BCE），希斯蒂埃乌斯（Histiaeus）剃光了他最信任的奴隶的头，并在上面刺了一个信息。然后等他的头发重新长出来，再把他送到阿里斯塔戈拉斯（Aristagoras）那里。通过这次通信，希斯蒂埃乌斯引发了一场反抗波斯人的起义 [^2, p81]。

希罗多德讲述的另一个早期使用隐写术的例子是德玛拉图斯（Demaratus）。他将信息刻在木板上，然后用蜡覆盖，以此来隐藏信息 [^2，p81-82]。
#### 1.1.2 隐分析
隐分析（Steganalysis）是 "检测和解码以隐图方式隐藏的信息的艺术"[3, p546]。隐写分析的最基本目标是确定信息中是否可能有嵌入的 "有效载荷"，在这一点上，隐写术的主要目标--秘密通信--就落空了。隐写分析的进一步目标通常包括确定用于隐藏数据的手段、估计隐藏数据的数量，以及最终恢复这些隐藏数据。

隐写分析一般由一方进行，通常称为管理员（*warden*）。管理员可以自由访问所有交换的信息。管理员可分为两大类[^4]：

**被动管理员**

检查所有正在交换的数据，但除非有可疑之处，否则不会干预。

**主动管理员**

在信息通过其网络时动态更改所有数据。这样做的目的是试图删除任何隐藏内容，而不管他是否检测到任何可疑内容。这在某些情况下是完全合理的，二战期间也有类似的做法。许多士兵写给家人的信都被拆开并重新措辞，以传递相同的信息，破坏任何隐藏的信息。传说有一封这样的信 在回信时被问了一个露骨的问题："父亲是死了还是去世了？"[^5].
### 1.2 可扩展消息和在场协议（XMPP）
可扩展信息传送和存在协议是一个 XML 流协议，允许交换信息和存在。（在线状态显示用户的状态，是在线、不在线还是忙碌。）XMPP 的分散架构允许每个组织控制自己的域，同时仍允许域间进行通信。该协议依靠 XML 流作为传输数据的媒介。这就允许任何有关方面在现有协议的基础上开发定制功能。这种可扩展性和分散性使该协议在现实生活中非常灵活和有吸引力。大量可用的客户端、服务器和代码库使其成为公司在应用基础上更受欢迎的选择之一，并使其成为事实上的行业标准。

一些规模较大、知名度较高的 XMPP 部署包括

- Google - Google Talk 服务基于 XMPP。（谷歌已开始逐步淘汰 XMPP，转而使用自己专有的 hangouts 协议。）
- Facebook - Facebook 于 2008 年宣布他们的聊天服务将支持 XMPP [6，7]。（2009 年，chat.facebook.com 被检测到运行修改版 ejabberd XMPP 服务器，用于连接 XMPP 客户端和 Facebook 服务器。）
- Cisco - Cisco 在其所有消息传递和出席技术中都使用了 XMPP [8]。
- LiveJournal - LiveJournal Talk 使用 XMPP 作为聊天和向 LiveJournal 服务发布更新的协议 [9]。
- BBC Radio LiveText - LiveText [10] 使用 XMPP 扩展 XEP-0124: Bidirectional-streams Over Synchronous HTTP[11] 将实时文本与其广播流一起推送。

XMPP 由两份 "互联网标准 "文件定义。它们是 RFC 6120: XMPP: Core [12]，定义了核心 XML 流技术；RFC 6121: XMPP: Instant Messaging and Presence [13]，定义了消息和在场协议。
## 2 拟议的隐蔽信道
隐蔽信道之所以被称为隐蔽信道，是因为它们是合法通信中的隐蔽通信途径。它们通常利用介质的某些特性（通常是高冗余度，但也有其他特性），使隐蔽通信的交换成为可能[14]。隐蔽信道有时会利用通信信道经常有较大开销这一事实，在许多实现中，需要不断传输元数据以保持信道畅通。

XMPP 就是这种通信协议的一个例子，它需要在每条信息中传输元数据以保持信道畅通。这些元数据是隐藏信息的方便之处。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbe’><body>Message.</body></message>
例 1：XMPP 消息。
```
例 1 显示的是一条非常简单的 XMPP 消息，没有隐藏数据。我们可以将其与下面的示例进行比较，后者说明了隐藏数据的交换。

下文将对我们发现并提出的隐蔽信道进行描述和分析：
### 2.1 类型属性
隐藏数据的一个地方是 "类型 "属性。服务器不会使用该属性，而是将其原封不动地传递给接收者。这使其成为存储信息的理想场所。
#### 2.1.1 更改 type 属性值的情况
由于服务器不会更改此属性的值，我们可以更改 type 属性值的大小写（从 "chat "改为 "CHAT "或 "ChAT"），以表示设置或清除位。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’CHAT’
id=’7df1ddbe’><body>Message.</body></message>
例 2：按类型属性情况编码的一个隐藏位
```
#### 2.1.2 更改 type 属性的值
在该属性中隐藏数据的另一种方法是使用 XMPP 规范中定义的五个有效值之一。这种方法允许我们在每条信息中最多编码 2 比特。不过，我们建议只使用其中的两个值 "normal"（正常）和 "chat"（聊天）来为每条信息编码 1 比特。这是因为其他两个选项 "错误 "和 "标题 "在聊天信息中并不常用，很容易引起怀疑。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’normal’
id=’7df1ddbe’><body>Message.</body></message>
例 3：在类型属性值中编码了一个隐藏位。
```
#### 2.1.3 存在类型属性
由于 XMPP 规范（RFC 6121 [13，第 5.2.2 小节]）并未强制使用该属性，因此我们可以在每个 XMPP 消息中用另一个比特来表示是否存在该属性。
```
<message from=’adam@test.com’ to=’bart@test.com’
id=’7df1ddbe’><body>Message.</body></message>     
例 4：在在场类型属性的情况下对一个隐藏位进行编码。
```
### 2.2 id 属性
每个 XMPP 消息都允许有一个 id 属性。id属性可以是任何字符串，用于识别单个信息标签。它通常使用某种计数器（数字或字母数字，通常用十六进制编码）来实现，在每条信息发出后都会递增，为每条信息提供唯一的标识符。如下所述，这种通道也可以通过多种方式进行隐写：

#### 2.2.1 最小有效位
对这些标识符的唯一要求是每个标识符都应是唯一的。XMPP 规范没有规定它们必须出现的顺序。我们可以使用该标识符的最小有效位作为隐藏数据的额外通道。

实现这一点的简单方法是丢弃标识符值，直到其 LSB 与要发送的数据相匹配。仅仅这样做可能会让人觉得可疑，因为 id 值会随着每条信息随机变化。一种更隐蔽、更安全的实现方法是，使用丢弃的标识符值向第三方（不参与隐写交换的诱饵 XMPP 客户端）发送信息。这样，每个标识符只使用一次，这种隐秘通信就更难被发现。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbf’><body>Message.</body></message>     
例 5：使用 id 属性的 LSB 一个隐藏位。
```
#### 2.2.2 大小写
利用 id 属性的另一个可能渠道是利用文本字符串中固有的冗余，在本建议中就是利用其大小写。由于 ID 是作为字符串实现的，我们可以将某些数据编码为字符串中各个字符的大小写。我们可以利用这一点为每条信息编码多个比特。

不建议在同一个 ID 中使用多种情况，因为这可能会引起不必要的怀疑。因此，最好在整个 ID 中使用一种情况，并在每条信息中只编码一位数据。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7DF1DDBE’><body>Message.</body></message>
例 6：id 属性中的一个编码位。
```
### 2.3 xml:lang 属性
由于 XMPP 依赖 XML 作为其核心技术，因此不同 XML 规范中的一些元数据属性也进入了 XMPP。"xml:lang" [15] 属性就是这些属性之一，用于确定信息所使用的语言，如英语 (en) 或法语 (fr)。可以使用该属性的频道有
#### 2.3.1 xml:lang 属性的存在
该属性是另一个可选属性，它是有效 XMPP 消息的一部分，使用该字段的存在与否可以隐藏一位数据。在本研究中开发的原型工具使用该通道对一位数据进行编码。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbe’><body xml:lang=’en’>Message.</body></message>     
例 7：在 xml:lang 属性存在的情况下编码的一个比特。
```
#### 2.3.2 xml:lang 属性的值
通过为该属性中的语言代码赋值，可以扩展该频道。这样，利用代表大致相同语言的语言代码（en-GB、en-US 等）的冗余，我们就可以对一些数据进行编码。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbe’><body xml:lang=’en-GB’>Message.</body>
</message>     
例 8：xml:lang 属性值中的一个隐藏位。
```
### 
