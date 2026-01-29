# 使用可扩展消息和在场协议（XMPP）的隐写术
> [Steganography using the Extensible Messaging and Presence Protocol (XMPP)](https://doi.org/10.48550/arXiv.1310.0524).
>
> by [Reshad Patuck](https://arxiv.org/search/cs?searchtype=author&query=Patuck,+R), [Julio Hernandez-Castro](https://arxiv.org/search/cs?searchtype=author&query=Hernandez-Castro,+J). *School of Computing, University of Kent, Cornwallis South Building, Canterbury, CT2 7NF, UK*
### 摘要
我们首次提出了在可扩展消息和在场协议（XMPP）中隐藏数据的不同机制。这是一种非常流行的即时通讯协议，被 Google Talk、Cisco、LiveJournal 等许多通讯平台所使用。本文介绍了如何在不引起任何中间人怀疑的情况下，从一个 XMPP 客户端向另一个客户端发送秘密信息。与其他试图在即时信息内容中隐藏数据的相关工作不同，本文描述的方法主要侧重于使用底层协议作为隐写术的手段。通过这种方法，我们提供了一种更强大的数据隐藏手段，并对其一般安全性进行了初步分析，特别是针对基于熵的隐写分析。

**关键词**：隐写术、隐分析、隐蔽信道、数据隐藏、XMPP、即时信息、协议
## 1 简介
### 1.1 隐写术
隐写术（Steganography）是一门交换隐藏消息的科学和艺术，这种交换方式除了发送者和预期接收者之外，没有人会怀疑任何隐藏数据的存在。Steganography 一词源于希腊语，*Steganos* 意为 "覆盖或保护"，*Graphei* 意为 "书写"。

与密码学不同，隐写术只涉及隐藏信息本身的存在。隐写术的目的不是使数据无法读取，而是完全隐藏这些数据。理想的隐写信息看起来与普通的干净信息完全相同，不会引起任何怀疑[^1]。
#### 1.1.1 历史
希腊历史学家希罗多德（Herodotus）向我们讲述了爱奥尼亚（Ionia）的起义（约公元前 499 年，ca. 499 BCE），希斯蒂埃乌斯（Histiaeus）剃光了他最信任的奴隶的头，并在上面刺了一个信息。然后等他的头发重新长出来，再把他送到阿里斯塔戈拉斯（Aristagoras）那里。通过这次通信，希斯蒂埃乌斯引发了一场反抗波斯人的起义 [^2],[p81]。

希罗多德讲述的另一个早期使用隐写术的例子是德玛拉图斯（Demaratus）。他将信息刻在木板上，然后用蜡覆盖，以此来隐藏信息 [^2],[p81-82]。
#### 1.1.2 隐分析
隐分析（Steganalysis）是 "检测和解码以隐图方式隐藏的信息的艺术"[^3],[p546]。隐写分析的最基本目标是确定信息中是否可能有嵌入的 "有效载荷"，在这一点上，隐写术的主要目标--秘密通信--就落空了。隐写分析的进一步目标通常包括确定用于隐藏数据的手段、估计隐藏数据的数量，以及最终恢复这些隐藏数据。

隐写分析一般由一方进行，通常称为监管员（*warden*）。监管员可以自由访问所有交换的信息。监管员可分为两大类[^4]：

**被动监管员**

检查所有正在交换的数据，但除非有可疑之处，否则不会干预。

**主动监管员**

在信息通过其网络时动态更改所有数据。这样做的目的是试图删除任何隐藏内容，而不管他是否检测到任何可疑内容。这在某些情况下是完全合理的，二战期间也有类似的做法。许多士兵写给家人的信都被拆开并重新措辞，以传递相同的信息，破坏任何隐藏的信息。传说有一封这样的信 在回信时被问了一个露骨的问题："父亲是死了还是去世了？"[^5].
### 1.2 可扩展消息和在场协议（XMPP）
可扩展信息传送和存在协议是一个 XML 流协议，允许交换信息和存在。（在线状态显示用户的状态，是在线、不在线还是忙碌。）XMPP 的分散架构允许每个组织控制自己的域，同时仍允许域间进行通信。该协议依靠 XML 流作为传输数据的媒介。这就允许任何有关方面在现有协议的基础上开发定制功能。这种可扩展性和分散性使该协议在现实生活中非常灵活和有吸引力。大量可用的客户端、服务器和代码库使其成为公司在应用基础上更受欢迎的选择之一，并使其成为事实上的行业标准。

一些规模较大、知名度较高的 XMPP 部署包括

- Google - Google Talk 服务基于 XMPP。（谷歌已开始逐步淘汰 XMPP，转而使用自己专有的 hangouts 协议。）
- Facebook - Facebook 于 2008 年宣布他们的聊天服务将支持 XMPP [^6],[^7]。（2009 年，chat.facebook.com 被检测到运行修改版 ejabberd XMPP 服务器，用于连接 XMPP 客户端和 Facebook 服务器。）
- Cisco - Cisco 在其所有消息传递和出席技术中都使用了 XMPP [^8]。
- LiveJournal - LiveJournal Talk 使用 XMPP 作为聊天和向 LiveJournal 服务发布更新的协议 [^9]。
- BBC Radio LiveText - LiveText [^10] 使用 XMPP 扩展 XEP-0124: Bidirectional-streams Over Synchronous HTTP[^11] 将实时文本与其广播流一起推送。（BBC LiveText 服务就是 XMPP 用于广播数据而非聊天的一个例子。）

XMPP 由两份 "互联网标准 "文件定义。它们是 RFC 6120: XMPP: Core [^12]，定义了核心 XML 流技术；RFC 6121: XMPP: Instant Messaging and Presence [^13]，定义了消息和在场协议。
## 2 拟议的隐蔽信道
隐蔽信道之所以被称为隐蔽信道，是因为它们是合法通信中的隐蔽通信途径。它们通常利用介质的某些特性（通常是高冗余度，但也有其他特性），使隐蔽通信的交换成为可能[^14]。隐蔽信道有时会利用通信信道经常有较大开销这一事实，在许多实施方案中，需要不断传输元数据以保持信道畅通。

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
由于 XMPP 规范（RFC 6121 [^13],[第 5.2.2 小节]）并未强制使用该属性，因此我们可以在每个 XMPP 消息中用另一个比特来表示是否存在该属性。
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
由于 XMPP 依赖 XML 作为其核心技术，因此不同 XML 规范中的一些元数据属性也进入了 XMPP。"xml:lang" [^15] 属性就是这些属性之一，用于确定信息所使用的语言，如英语 (en) 或法语 (fr)。可以使用该属性的频道有
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
### 2.4 主体内容
XMPP 消息的正文包含消息的实际内容。

下面我们将列出并介绍几种最实用的隐蔽信道，它们都采用信息内容作为载体。

#### 2.4.1 前导和尾部空间
前导和尾部空间冗余是一种为每条信息的每个主体元素编码 2 比特数据的方法。具体做法是首先修剪（修剪是一种字符串操作技术，用于删除任何前导空格和尾部空格。）信息的正文，然后在信息的开头和/或结尾添加一个空格字符，每个空格字符编码一个比特的数据，这种方法与众所周知的隐写工具 SNOW [^16]非常相似。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbe’><body> Message.</body></message>
例 9：一个比特隐藏在前导空格中。
```
#### 2.4.2 用同义词替换词语
在信息正文中隐藏数据的另一种方法是用同义词或缩写词替换词语。这就利用了用于编码数据的语言中的冗余。

这一功能可以通过单词及其相应同义词的字典来实现。然后，我们可以根据在给定时间内从同义词列表中使用的等价词来对某些比特进行编码。这种实现方式只能通过对每条信息中使用的语言进行启发式检查来检测[^17]，但任何此类技术都很可能会出现许多误报，而且总体准确性较差。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbe’><body>Msg.</body></message>     
例 10：用信息缩写代替信息隐藏一个比特。
```
#### 2.4.3 拼写错误
拼写错误在快速输入的聊天信息中很常见。我们可以利用一些众所周知的错误来编码单词拼写是否正确的数据。

该信道的实现方式与上文所述的同义词信道十分相似，也是使用字典进行数据编码，并不具备任何微不足道的高精度隐写技术。
```
<message from=’adam@test.com’ to=’bart@test.com’ type=’chat’
id=’7df1ddbe’><body>Mesage.</body></message>     
Example 11: One bit coded by a spelling mistake.
```
## 3 实现隐蔽信道
我们创建了一个名为网址 [StegMPP](http://patuck.github.io/StegMPP/) 的概念验证工具，用于在前几节所述的一些隐蔽信道上实施隐写术，并测试其容量和安全性。它有一个简约的图形界面，只实现了收发信息所需的 XMPP 严格比特。[图 1](https://ar5iv.labs.arxiv.org/html/1310.0524/assets/StegMPP.png)、[图 2](https://ar5iv.labs.arxiv.org/html/1310.0524/assets/Connection.png) 和[图 3](https://ar5iv.labs.arxiv.org/html/1310.0524/assets/Steganography.png) 展示了 StegMPP 的用户界面。

![1](https://ar5iv.labs.arxiv.org/html/1310.0524/assets/StegMPP.png)
图 1：StegMPP 用户界面
![2](https://ar5iv.labs.arxiv.org/html/1310.0524/assets/Connection.png)
图 2：连接设置用户界面
![3](https://ar5iv.labs.arxiv.org/html/1310.0524/assets/Steganography.png)
图 3：隐写设置用户界面

我们工具的隐写术子系统将用户数据加密并嵌入不同的隐蔽信道。具体如下
- id 属性的值。
- xml:lang 属性的存在。
- 类型属性的存在。
- 正文中是否存在前导空格。
- 正文中是否存在尾部空格。
- type 属性 case.（由于下一节所述的已知安全漏洞，StegMPP 最终删除了这一属性。）
## 4 隐蔽信道分析
这些隐蔽信道的隐写分析是通过模拟监管员进行的。主动监管员并不适合这里描述的某些隐蔽信道，因为我们发现，在某些信道中任意和/或随机改变值会妨碍普通用户的通信。因此，在下文中，我们倾向于使用被动式监管员来模拟现实环境，以监控所交换的信息。

根据是否考虑到协议的先前状态，监管员可进一步分为两种：
### 4.1 无状态监管员
无状态监管员是一种独立处理每条信息的监管员，它不考虑任何之前的信息。这类监管员很容易实现，因为它只需将每条信息与静态定义列表进行比较即可。

不过，这种保护机制只能检测到非有效 XMPP 消息的隐藏数据。由于 RFC [^13],[subsection 5.2.2]规定了特定的值，而且所有值都是小写，因此可以检测到类似 "type "属性的信道。这使得查找隐藏通信变得轻而易举，因为 "类型 "属性中包含大写值的任何信息都会被立即归类为可疑信息。
### 4.2 有状态监管员
有状态监管员在许多方面都比无状态监管员更强大。它可以跟踪以前的信息，并将所有新信息与以前的信息列表进行比较。这使得捕获隐藏内容变得更容易，因为安全监管员不是根据定义列表进行匹配，而是根据协议的先前状态进行匹配。
|隐蔽信道|无状态监管员|有状态监管员
|-|-|-
|类型属性的存在|无|有
|ID 属性的值|无|是（如果被丢弃的 ID 被用于发送其他客户端信息，则可能无法检测到。）
|xml:lang 属性的存在|无|是
|头部空间|无|无
|尾部空间|无|无
|属性类型案例（该属性已从 StegMPP 中删除，因为它甚至能被无状态监管员检测到。）|是|是

表 1：隐蔽信道及其可探测性。
## 5 结果
对两种类型的守护程序进行模拟的结果表明，StegMPP 实现的大多数信道都不会被无状态守护程序检测到。它还表明，其中一些信道可以被精心设计的有状态安全卫士检测到。然而，在繁忙的 XMPP 服务器上监控所有连接的有状态警报器的实现并非易事，它需要大量的计算资源，并能处理可能出现的大量误报。

第 1 页的表 1 列出了测试过的隐蔽信道（每个信道对每条信息隐藏一个比特），以及它们对无状态和有状态警报器的安全性。
### 5.1 统计分析
我们使用 ent（一个实施一系列统计测试的工具）计算每个字节的熵，并找出正常（干净）XMPP 会话与使用隐蔽信道交换数据的会话之间是否存在显著的统计差异。分析结果见第 2 页的表 2。
|频道|熵|差异
|-|-|-
|控制|4.976370|
|类型属性的存在|4.974257|-0.002113
|ID 属性值|4.980152|0.003782
|存在 xml:lang 属性|4.989826|0.013456
|头部空间|4.975970|-0.000400
|尾部空间|4.975430|-0.000940

表 2：隐蔽信道的熵值比较。

对表 2 的详细分析清楚地表明，由于清洁（控制）交换所对应的值与其他隐藏交换所对应的值之间的差异微乎其微，因此任何基于熵指标对拟议技术进行的统计隐写分析都必须极其精确。

隐藏内容信道的熵值并不总是高于干净信道的熵值，这也说明了这一困难。这种情况在不同的隐写设置中经常出现，但在这里，我们在最后一列（差值）中采用了交替符号。

尽管如此，这些数字也表明，基于 xml:lang 属性的隐蔽信道似乎比其他信道更容易被检测到，因此在高安全性环境中不建议使用这种信道。

当然，这本身并不能证明安全性，但至少表明本文提出并在附带工具中实现的技术对封面媒体的修改非常轻微，这给我们一些启示，即未来任何基于熵的隐写分析都需要大量数据和计算能力才能成功。
## 6 结论
本研究首次提出了一种利用 XMPP 进行隐写通信的新方法。文中详细讨论了一些新的、有前景的隐蔽信道。这些信道大多利用了底层协议中的冗余，而不是信息内容。仿真表明，这些信道中的大多数都能不被无状态监管者发现，但可能会被有状态监管者发现。此外，我们还提出了一些微妙的方法来强化这些信道的子集，以抵御有状态安全卫士的攻击。

我们的概念验证工具 StegMPP 附带了本文所述的许多建议功能，可在以下网址免费下载：http://patuck.github.io/StegMPP/
### 6.1 未来工作
为了开发更安全的实施方案，我们建议进一步研究如何为有状态安全卫士加固隐写分析。实现这一目标的方法之一是交替使用每条信息中的信道。我们相信这个想法很有前途，并计划在不久的将来进行更详细的研究。我们还想找到一种方法来提高能够检测这些隐蔽信道的有状态守护程序的效率，并开发一种隐写分析工具来检测 XMPP 信息中是否存在隐藏信息。这种工具可能会对机构、大型企业和政府组织有用，特别是在早期检测内部威胁和数据泄漏方面。
[^1]:Neil F Johnson and Sushil Jajodia.Exploring steganography: Seeing the unseen.*IEEE computer*, 31(2):26–34, 1998. URL: http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4655281.
[^2]:David Kahn.*The codebreakers : the story of secret writing ; [the comprehensive history of secret communication from ancient times to the Internet]*.Scribner, New York, NY, 1996.ISBN: 0684831309, 9780684831305.
[^3]:I. J. Cox, M. L. Miller, J. A. Bloom, J. Fridrich, and T. Kalker. *Digital watermarking and steganography*.Morgan Kaufmann Publishers Inc, 2nd edition, 2007.ISBN: 0123725852. URL: http://www.dawsonera.com/depp/reader/protected/external/AbstractView/S9780080555805.
[^4]:Ismail Avcibas, Nasir Memon, and Bülent Sankur.Steganalysis using image quality metrics.*Image Processing, IEEE Transactions on*, 12(2):221–229, 2003.
URL: http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1192983.
[^5]:R.J. Anderson and Fabien A P Petitcolas.On the limits of steganography.*Selected Areas in Communications, IEEE Journal on*, 16(4):474–481, 1998.ISSN: 0733-8716.DOI: 10.1109/49.668971. URL: http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=668971.
[^6]:Using Facebook Chat via Jabber - Facebook Developers Blog. URL: http://developers.facebook.com/blog/post/110/.
[^7]:Facebook Chat API - Facebook Developers. URL: http://developers.facebook.com/docs/chat/.
[^8]:Jabber Technology - Cisco Systems. URL: http://www.cisco.com/web/about/ac49/ac0/ac1/ac258/JabberInc.html.
[^9]:Frequently Asked Question #270 - What is LJ Talk? – LiveJournal. URL: http://www.livejournal.com/support/faq/270.html.
[^10]:BBC - Radio Labs: LiveText-via-IP upgrade and other synchronously delivered content. URL: http://www.bbc.co.uk/blogs/radiolabs/2009/11/pushfeeds.shtml.
[^11]:Ian Paterson, Dave Smith, Peter Saint-Andre, and Jack Moffitt.XEP-0124: Bidirectional-streams over synchronous HTTP (BOSH), 2010.
[^12]:Peter Saint-Andre.*RFC 6120: Extensible Messaging and Presence Protocol (XMPP): Core*.Internet Engineering Task Force, March 2011.
URL: http://www.ietf.org/rfc/rfc6120.txt.
[^13]:Peter Saint-Andre.*RFC 6121: Extensible Messaging and Presence Protocol (XMPP): Instant Messaging and Presence*.Internet Engineering Task Force, March 2011. URL: http://www.ietf.org/rfc/rfc6121.txt.
[^14]:Robert C. Newman.Covert computer and network communications.In *Proceedings of the 4th annual conference on Information security curriculum development*, page 12. ACM, 2007. URL: http://dl.acm.org/citation.cfm?id=1409922.
[^15]:*Language tags in HTML and XML*, September 2009. URL: http://www.w3.org/International/articles/language-tags/.
[^16]:M. Kwan.How SNOW works, 1996. URL: http://www.darkside.com.au/snow/description.html.
[^17]:M.H. Shirali-Shahreza and M. Shirali-Shahreza.Text Steganography in chat.In *Internet, 2007. ICI 2007. 3rd IEEE/IFIP International Conference in Central Asia on*, pages 1–5, 2007.DOI: 10.1109/CANET.2007.4401716. URL: http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4401716.
[^18]:Shingo Inoue, Kyoko Makino, Ichiro Murase, Osamu Takizawa, Tsutomu Matsumoto, and Hiroshi Nakagawa.A proposal on information hiding methods using XML.In *Proceedings of the first NLP and XML Workshop*, pages 55–62, 2001. URL: http://www.takizawa.ne.jp/nlp_xml.pdf.
[^19]:Jonathan Cummins, Patrick Diskin, Samuel Lau, and Robert Parlett.Steganography and digital watermarking.*School of Computer Science, The University of Birmingham*, 14:60, 2004. URL: http://www.cs.bham.ac.uk/~mdr/teaching/modules03/security/students/SS5/Steganography.pdf.
