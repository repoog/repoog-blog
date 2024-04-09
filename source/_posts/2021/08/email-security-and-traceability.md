---
title: 邮件安全防护及溯源
date: 2021-08-30 14:18:40
author: repoog
summary: 邮件安全是企业安全的基础安全的重要环节，但涉及到邮箱安全设置的配置却不仅一点，可能会由于管理人员或运维人员对于邮箱安全的不熟悉，导致邮箱并未真正得到安全保障。本文介绍的是邮件安全中的安全防护设置以及基于邮件头的邮件发件来源溯源原理。
comments: true
tags:
  - DKIM
  - DMARC
  - SPF
  - 邮件伪造
  - 邮件安全
  - 邮件溯源
categories:
  - 邮件安全
---

电子邮件涉及到三种传输协议，SMTP、POP3和IMAP。其中，SMTP是一个相对简单的基于文本的协议，在该协议指定了一条消息的一个或多个接收者，然后将消息传输，所以SMTP是一个“推送”协议，可以在多个SMTP服务器之间推送邮件信息，它无法做到从远程服务器上“拉取”信息，邮件客户端从服务端“拉取”邮件需要使用POP3或IMAP协议。SMTP服务器通常使用的端口包括TCP的25端口（部分邮件服务商会关闭该端口，停止使用明文方式传输邮件），或者TLS的587端口，又或者SSL的465端口。

邮件发送涉及到的基本概念包括：

* MUA：Mail User Agent，邮件使用代理，比如Foxmail等，负责接收邮件信息，提供邮件读写环境，不负责邮件传输；
* MSA：Mail Submission Agent，邮件提交代理，通常和MTA是一个程序或服务，并部署在相同的服务器上，也可以分开部署，MSA接收到邮件后会将邮件转发给MTA；
* MTA：Mail Transfer Agent，邮件传输代理，负责传输邮件的程序，比如Sendmail、Postfix等，MTA收到MSA邮件后，会根据邮件的目的地址做判断，将邮件传输到目的地；
* MDA：Mail Deliver Agent，邮件传送代理，MTA收到邮件后如果判断邮件是本域下的，则把邮件转到MDA处理，MDA再将邮件转发给收件人；

![邮件发送过程](images/2021/08/send_mail.png '邮件发送过程')

一封邮件的发送过程如下：

1. MUA通过SMTP协议发送邮件到MSA；
2. MSA推送邮件到MTA；
3. MTA使用DNS协议查询收件人域名的MX记录，MX记录包含目的地址的MTA，并根据该地址发送邮件，收到邮件的MTA可能是目标MTA，也可能是开放邮件中继（open mail relay），如果是后者，则需要进一步转发；
4. MTA收到邮件内容后转给MDA进行本地投递；
5. MDA会将邮件信息存储，方便经过认证的MUA读取；
6. 邮件客户端使用IMAP或POP3协议拉取MDA中的邮件信息；

最初的SMTP协议只支持未经认证的、未加密的7位ASCII文本通信，容易受到中间人攻击、邮件欺骗和垃圾邮件的影响，并要求任何二进制数据在传输之前被编码为可读文本，相比早期的SMTP协议，现在的SMTP协议增加了身份认证、加密、二进制数据传输等扩展。SMTP服务器曾经默认配置是作为开放邮件中继（open mail relay），开放邮件中继允许任何人通过它发送邮件，由于垃圾邮件和蠕虫利用，后开放邮件中继相继被关闭，或被邮件服务商加入黑名单。

发送方和接收方SMTP服务器创建连接后，会创建一个SMTP会话，可以使用telnet命令创建SMTP连接，下图是使用telnet命令发送一封正常的邮件：

![telnet发送邮件](images/2021/08/telnet_send_mail.png 'telnet发送邮件')

客户端通过使用EHLO问候语来了解SMTP服务器所支持的选项，当服务器不支持EHLO命令时会退回到使用更早的命令HELO，其中SIZE表示SMTP服务器可以支持的最大信息量，单位是字节。

上图中，有四点值得注意：

* EHLO指定的域名可以与MAIL FROM:域名不相同，前者指定的是发件SMTP服务器域名，但可以随意填，后者才是发送邮件的真正地址，且<和>必填；
* AUTH认证的邮箱地址必须与MAIL FROM:的邮箱相同，否则SMTP服务器无法发送邮件，并提示账户错误或不在认证的邮箱域下；
* MAIL FROM:可以与From:不同，前者是真正的邮件发送地址，后者是邮件内容显示的发件地址；
* RCPT TO:可以与To:不同，前者是真正的邮件接收地址，后者是邮件内容显示的收件地址；

邮件伪造的基础便来自以上四点，但由于邮件收发服务器（MSA/MTA）都由邮件服务商负责，因此不同的邮件服务商可以配置服务器处理邮件的方式，并帮助用户识别邮件伪造，比如MAIL FROM:如果与From:内容不同，则在显示邮件内容时，发件人会提示邮件是代发，提示用户注意识别邮件真实性，但并不是每个邮件服务商都这么贴心，贴心的服务商对于RCPT TO:和To:不一致的问题也并不会提示。

由于邮件伪造、邮件欺诈等问题，邮件安全防护的措施目前有三种：SPF、DKIM和DMARC。

### **SPF（Sender Policy Framework，发件人策略框架）**

![SPF工作原理](images/2021/08/SPF.png 'SPF工作原理')

发件人策略框架的标准文件是RFC7208，是最早期广泛部署的电子邮件授权技术之一，在SPF之前，发送主机对邮件中可以使用的MAIL FROM值没有任何限制，可以随意设定自己的发件地址。SPF用于为特定邮箱域名提供可以信任的邮件服务器的主机列表，收件服务器由此可以确认邮件是由被授权的邮箱服务器发出，防止有人伪冒身份网络钓鱼或发送垃圾电邮。

SPF使用DNS记录存储可以信任的邮件服务器列表，域名管理员可以发布一个DNS的TXT记录，指定哪些主机可以在该域名下发送邮件。比如查询qq.com域名的SPF记录：

![qq.com的SPF记录](images/2021/08/spf_qq_com.png 'qq.com的SPF记录')

SPF是迭代结构的信息，include部分可以再包含其他域名，迭代查询其TXT记录，收件邮箱服务器会查询发件邮箱服务器地址是否在SPF记录中，并根据SPF最末的策略处理邮件。

SPF的处理策略包括：

* +：放行，如果没有明确指定限定词，则为默认值；
* \-：硬拒绝，直接拒绝来自未经授权主机的邮件，并发回退信邮件；
* ~：软拒绝，邮件被接受，但应被标记为垃圾邮件；
* ?：中性，不考虑邮件是否被接受，该策略是默认策略；

上图中的SPF策略是硬拒绝，如果收件服务器检查邮件发送服务器地址不在图中的SPF记录中，则会退回邮件。但邮件自动转发是例外，邮件自动转发要求包括发件人和收件人信息均不变，比如Alice通过alice@a.com发邮件给Bob的bob@b.com邮箱，Bob设置了邮件自动转发至bob@bbox.com，则邮件会由b.com的收件服务器自动转发至bob@bbox.com，bbox.com的收件服务器收到邮件后，由于发件服务器是b.com，而发件地址是alice@a.com，因此无法通过a.com的SPF记录检查，所以SPF无法解决邮件自动转发的问题。

SPF的另一个不足在于，include的记录可能会发生变更，且邮件服务商并不会通知变更，如果邮件服务商的记录存在疏漏，比如包含未信任的服务器地址，会影响所有客户及用户。另外，根据SPF标准，SPF的DNS迭代查询次数最多只有10次，如果需要10次以上的DNS查询来解析一条SPF记录，则之后的记录被认为无效。因此如果涉及庞杂的邮件服务系统，SPF有扩展性不足问题。

对于SPF常见的误解是，能够彻底解决邮件伪造问题，但实践发现并非如此，即便邮箱域名设置了SPF记录，邮件依然可以被伪造。原因是，邮件服务器收到邮件时，SPF检查并不对邮件头中From:部分进行验证，而是检查返回路径（Return-Path）来验证发件服务器地址是否在SPF记录中，返回路径是收件服务器用来通知发件服务器邮件发送问题（比如退信）的邮件域名，通常与EHLO指定的域名相同。因此，无论From:是真是假，邮件都可以通过SPF检查，而From:会以“发件人”出现在邮件内容中。部分邮件服务商会将Return-Path和From:区分显示，以帮助用户识别邮件伪造，比如Gmail邮箱会在邮件头中分别显示Return-Path、Received-SPF，以帮助检查与From:中域名是否一致。

![Gmail邮件源码的return-path](images/2021/08/return_path_example.png 'Gmail邮件源码的return-path')

### **DKIM（Domain Keys Identified Mail，域名密钥识别邮件）**

![DKIM工作原理](images/2021/08/DKIM.png 'DKIM工作原理')

域名密钥识别邮件标准文件是RFC6376，是一种让邮件服务器以加密方式验证与某个域名相关电子邮件真实性的方法，工作原理是在每封发出的邮件头中添加一个数字签名，该数字签名与邮箱域名相关，且在邮箱域名DNS中的TXT记录有签名的公钥。与SPF不同的是，DKIM的TXT记录需要知道DKIM域名的选择器（selector）才能查询。通常情况下，在邮件源码的DKIM-Signature头中，可以根据s=和d=来知道要解析的域名地址，“s=”表示选择器，“d=”表示域名，查询的DKIM域名是“s=.\_domainkey.d=”。比如下图中，s1024是选择器，newsletter.aliyun.com是邮箱域名。

![邮件源码中的DKIM签名](images/2021/08/dkim_sign_aliyun.png '邮件源码中的DKIM签名')

![DKIM的公钥](images/2021/08/dkim_dns_aliyun.png 'DKIM的公钥')

DKIM签名是由发件服务器的私钥完成签名，选择器是随机的文本，也是发件服务器选定，收件服务器收到邮件后，根据邮件头的签名信息获取DNS解析的域名，再进行DNS查询来查看公钥校验签名，如果签名校验通过，则表示邮件合法。

DKIM的DNS记录中，k是算法名称，p是RSA算法的公钥。邮件头的DKIM-Signature中，b=是h=标签中所列字段（上图中包括Date、From、To、Message-ID、Subject、MIME-Version、Content-Type）的签名信息，签名使用RSA-SHA256算法，再用BASE64编码，bh=是邮件内容的签名。

因为DKIM签名不涉及主机名、IP地址等发件地址信息，但又包含From:和To:发信人和收信人信息，因此在邮件自动转发问题中，可以用于检查邮件内容的真实性，弥补SPF的不足，但原理上只能支持发件邮箱域名下的邮件签名校验，所以无法保证邮件来自域名下的发信人邮箱，即依然没有解决邮件伪造问题，也无法解决邮件的不可抵赖问题，但这个可以通过PGP解决。

### **DMARC（Domain-based Message Authentication, Reporting & Conformance，基于域的消息认证、报告和一致性）**

基于域的消息认证、报告和一致性的标准文件是RFC7489，是一个建立在SPF和DKIM之上的政策引擎，指示邮件服务器如何处理不能通过SPF、DKIM或两者的邮件，还定义了如何向邮箱域名管理员报告这类邮件的失败情况，以帮助管理员调查为什么有效的邮件不能通过SPF或DKIM，或者了解试图冒充域名的人。和SPF、DKIM一样，DMARC同样依赖于在DNS中的策略记录。

![DMARC策略](images/2021/08/dmarc_tesla.png 'DMARC策略')

上图中标签的含义如下：

* p：应用于邮件的策略。上图的策略是quarantine（隔离），即将邮件放入垃圾邮件中。其他的值是：reject（拒绝），即拒收邮件，或者none（无），即什么都不做；
* pct：该策略是影响邮件的百分比。上图中，100%不符合DMARC策略的邮件应该被拒绝。当部署一个DMARC策略时，建议该值可以从0%慢慢增加到100%；
* rua：反馈报告的URI。上图中，邮件服务器的报告方式是发送电子邮件，其中包含所有未能通过DMARC策略的邮件。
* fo：定义DMARC未通过的含义。0代表SPF和DKIM都必须未通过，1代表SPF或DKIM都必须未通过，d代表只有DKIM未通过，s代表只有SPF未通过。1是最严格的，因为SPF或DKIM都可能未通过，且消息会被发送到rua中的地址。

根据上述原理，部署DMARC的前提是SPF或DKIM已经部署完毕。无论是SPF、DKIM还是DMARC，由于都依赖于DNS，因此DNS配置需要确保足够的安全，例如采用DNSSEC。

### **邮件溯源**

邮件源码中包含邮件发送服务器及开放邮件中继添加的各种字段信息，即邮件头。在遇到欺诈、广告或钓鱼邮件时，通过分析邮件头信息，可以帮助溯源邮件，包括转发的次数等等，对于自动发送的邮件可以溯源到邮件发送服务器，运气好的话，甚至可以溯源到具体的地理位置，对于只能溯源到邮件服务商的服务器地址，无法进一步溯源的，可以使用其他方式。

![邮件源码中的来源字段](images/2021/08/mail_source.png '邮件源码中的来源字段')

邮件头信息中与传送地址相关的头信息包括：

* Received：其中又包括by和from，前者是邮件接收服务器的IP或服务器名称，后者是发送或转发服务器的IP或服务器名称；
* X-Received：如果存在的话，表示初始发出邮件的服务器IP或服务器名称；

邮件头信息的阅读顺序是从下往上，因此如果存在多个Recevied头，邮件的收发顺序是从最后的Received到最开始的Recevied头。另外，Google有现成的工具可以帮助分析邮件头：https://toolbox.googleapps.com/apps/messageheader/analyzeheader 。

上图的信息经过分析如下：

![Google邮件头分析工具](images/2021/08/google_mail_header.png 'Google邮件头分析工具')