---
layout: post
title: "域名系统"
categories: network
tags: dns internet
permalink: /4/
---


域名系统，也就是 DNS，起源于 1983 年，它是互联网从 TCP/IP 发展到万维网的重要里程碑。DNS 为我们完成了从输入网址到获得网页内容过程中一个“理所应当”的步骤，我们甚至都不会意识到它的存在。在阅读了[Origins of the Domain Name System](https://ieeexplore.ieee.org/document/8700196)之后，来分享一下和 DNS 起源有关的故事。

## 阿帕网

DNS 起源于“名称到地址映射”需求规模的增长。二十世纪六十年代晚期，美国国防高级研究计划局（DARPA）资助开发了互联网鼻祖——阿帕网（ARPANET）项目。BBN （Bolt, Beranek and Newman Inc.）公司为这个网络提供了数据处理的协议，协议中用数字表示的主机地址与用文字表示的主机名做成一张表，完成了主机名和数字地址的关联。

连接到阿帕网的主机都有一个名字，当时将主机名 `UCLA` 代表加州大学洛杉矶分校（University of California, Los Angeles），`MIT` 代表麻省理工学院（Massachusetts Institute of Technology），`USC-ISI` 代表南加州大学资讯科学研究院（University of Southern California - Information Sciences Institute）。提供主机名和地址映射关系的表其实就是一个文本文件，这个文件被称为 `hosts.txt`，这个文件依旧能在我们现在使用的计算机操作系统中找到，一般被称为 `hosts` 文件。Windows 系统中，这个文件在 `C:\Windows\System32\drivers\etc\hosts`；Linux 系统中，这个文件一般在 `/etc/hosts`。提供映射关系的文本文件需要定期更新，维护和分发的工作由在 SRI-NIC 网络信息中心的 Jonathan Postel 组织，因而 Postel 也被称为阿帕网的“数字沙皇”。

阿帕网从 1969 年开始运行，不久后的 1971 年，Ray Tomlinson 实现了电子邮件系统，其中电子邮件的地址以 `@` 分隔用户名和主机名（例如 `cerf@usc-isi`），这种电子邮件地址表示方案沿用至今。

## Internet 和 DNS
随着阿帕网规模的扩大，DARPA 开始资助有关网络通信（network intercommunication，可能是 Internet 名字的来源）的研究。1973 年，Internet 的最初设计诞生。又经过十年的开发，1983 年 1 月 1 日，使用 TCP/IP 协议族、支持不同种类数据包传输的 Internet 开始正式运行。

此时，Postel 仍在负责管理地址和主机名的分配工作。二十世纪八十年代早期，Internet 范围逐渐扩大，定期分发 `hosts.txt` 文件的方式已经不能满足当时的需求，管理主机名和地址的系统必须有所改变以应对 Internet 的巨大规模。Paul Mockapetris 承担了设计下一代域名和地址映射系统的任务。起初，Postel 让 Paul 研究了几个方案，Paul 试图在其中找到一个尽量好的方案。后来他还是自己上手设计，对于刚获得博士学位的 Paul 来说，设计一个域名系统看起来是一个不错的小课题。在 MIT 就读时，Paul 设计过一个在众多小型计算机之间的分布式文件系统；在加州大学欧文校区就读时，Paul 设计了一个基于计算机名而不是地址的消息分发系统。有了这些这些经验，他设计使用多个冗余服务器保证来可用性，数据传输使用 UDP 协议。在随后的几年中，Paul 起草了两份关于实现 DNS 的 RFC 方案 RFC 882 和 RFC 883，不过这两份方案没有投入使用，RFC 1034 和 RFC 1035 改进了前两份方案，并成为了真正 DNS 技术规范。

在哪些名字可以作为顶级域名的问题上，一些“专家”认为应当域名层级应该按照网络实际结构来划分，或者只使用国家代码作为顶级域名，甚至认为 Internet 没有什么明显的商业用途而不需要 `.com` 这个顶级域名。

DNS 的分层结构非常适合分布式管理。`.com`、`.net`、`.org`、`.mil`、`.gov`、`.edu`、`.int`这些顶级域名（Top Level Domain，TLD）组成了 DNS 最初的“根域名”。顶级域名的管理者可以将二级域名（例如：`example.com`、`usc-isi.edu`、`nsf.gov`）的管理权委托给其他组织，三级域名同样可以委托。早期的主机名在 DNS 开始使用之后发生了一些变化，`UCLA` 变成了 `UCLA.EDU`，`SRI` 变成了 `SRI.COM`。

1986 年左右，第一台计算机开始在生产环境中依赖 DNS。当时 DNS 服务端软件的 UNIX 实现主要有两种，一个是斯坦福（Stanford）开发的 Druid，一个是伯克利（Berkeley）开发的 BIND，后来我们知道，BIND 使用更为广泛。

## 组织化管理
DNS 启用后，Postel 创建了 IANA 来负责分配域名和 IP 地址空间，并记录 TCP/IP 协议族中不同协议需要的参数名和参数值。慢慢地，Postel 逐渐把 13 个组织纳入到域名系统的“根域名服务器”的管理工作中。由 IANA 和 SRI 分配和指派的根域文件[^1]复制到了 13 个服务器中分散部署。后来，根域文件复制到了遍布于全世界的成百上千个服务器之中，成为根域名服务器的镜像[^2]。

二十世纪八十年代中后期，管理 Internet 的职责从 DARPA 转移到了多个美国机构，包括国家科学基金会（NSF）、国家航空航天局（NASA）和美国能源部（DOE），这些机构都曾参与将网络接入 Internet 的工作。NSF 继承了 SRI-NIC 管理 DNS 根域的工作。

1991 年，在阿帕网正式退役后，NSF 建立了互联网络信息中心（InterNIC），Network Solutions 公司承担管理根域和 3 个顶级域（`.com`，`.net`，`.org`）的工作。1992 年，NSF 认为已经不值得再为管理域名注册花费本属于科学研究的费用，开始允许 Network Solutions 公司向域名注册者收费，当时一个二级域名（例如：`foo.com`）的费用是 100 美元两年。

Internet 创建早期的实验阶段所需要的费用由美国政府和一些加入到网络基础设施建设的机构承担，当时域名和 IP 地址分配都是免费的。直到 1992 年，随着万维网的到来，域名注册量大大增加，域名注册的商机慢慢浮现。域名能被注册，也会过期，这就带来了潜在的问题：一个 URL 指向的资源在域名注册过期后将不能通过 DNS 解析到原 IP 地址，这个 URL 指向的资源将不复存在。如果这个 URL 中的域名被其他人注册，原 URL 就可能指向不同的资源。

1995 年左右，Network Solutions 公司被 SAIC 公司收购。1995 年 8 月，网景公司（Netscape Communications）上市，互联网泡沫由此开始。在互联网泡沫的顶峰，2000 年 3 月，SAIC 以 19.3 亿美元的价格将 Network Solutions 出售给了威瑞信（VeriSign）。

随着 Internet 的发展，域名管理称为了一桩大生意，Postel 和南加州大学的法律部门认为域名管理带来的风险和纠纷在逐渐增加，Postel 和他的团队觉得原来在与政府合约下执行的工作应当组织化。接着，各个利益方组建了 International Ad Hoc Committee (IAHC) 来讨论如何组织化域名管理的问题。起初，IAHC 提出在瑞士建立一个域名管理组织，这引起了美国国会的注意并使得白宫派出克林顿总统的顾问 Ira Manaziner 来处理该问题。最终，Manaziner 提出建立域名管理组织可行，但要由美国商务部下辖的国家电信管理局（NTIA）监管。1998 年，ICANN 赢得了 NTIA 运营 IANA 资格。遗憾的是，本能成为 ICANN 首席技术官的 Postel 在 ICANN 正式运营的前两周逝世。当时 ICANN 董事会由 Esther Dyson 任主席，任命 Michael Roberts 为第一任 CEO。ICANN 负责根域的管理，并向 VeriSign 提供更新根域文件的必要信息。VeriSign 也与 NTIA 达成了关于创建和分发主根域文件的协议。起初 VeriSign 运营 13 个根域名服务器中的 1 个，在一项收购之后 VerySign 又获得了 1 个根域名服务器的运营权。

## DNS 的安全问题
1992 年左右，Vint Cerf 和 Steve Crocker 等人发现了 DNS 的缓存投毒问题，这个问题甚至引起了当时的总统科学顾问的注意。其实在略早前，一位叫做 Steve Bellovin 的人就已经发现了这个问题，不过为了防止这个问题被用于恶意攻击，Bellovin 并没有立即发表他的研究成果。在 Vint 和 Crocker 的最初讨论中，他们想要用数字签名来解决这个问题，但要充分测试并得到广泛接受可不简单，从他们有想法到方案真正落地经过了近 **20** 年的努力。

他们的解决方案就是 DNSSEC（域名系统安全扩展）。Steve Crocker 的最初想法是确认域名和 IP 地址关联的正确性，并且 IP 地址是合法地分配到域名持有者手中的。利用域名的层级结构，如果要验证 `www.example.com` 和 `1.2.3.4` 之间的合法关系，必须确认关联这个域名和地址的人是上一级域名 `example.com` 的拥有者。`www.example.com` 和 `1.2.3.4` 关系的数字签名就必须由 `example.com` 的公钥签发，`example.com` 和其地址关系的签名必须由 `.com` 域签发，也就是每一级域名和地址的关系都要由上一级域名签发，顶级域名的合法性由根域名保证，这个方法和现在的 HTTPS 证书验证原理很像。

## DNS 的国际化
传统的域名允许使用的符号基于 ASCII 字符集，只能由字母、数字和连字符（`-`）组成，在 Tin Wee Tan 的推动下，使用不同语言的字符作为域名已经成为现实，其初衷是推动互联网在非英语国家的普及。试试在浏览器地址栏输入`新华网.中国`，看看能不能访问到。

国际化域名（Internationalized Domain Name，IDN）的实现从 1998 年开始，其思路是识别不同语言域名的编码，再转写为可以被传统 DNS 解析的 ASCII 字符域名。为了推动国际化域名在新千年实现，还成立了国际化域名协会，然而直到 2010 年，第一个 IDN 顶级国家域名才在 DNS 系统中正式应用。

## 一点感想
从 Internet 诞生至今也不过 40 年，不得不感叹计算机技术发展的速度。当初为解决网络规模扩张问题而出现的 DNS 成了现在互联网通信的指南针，而完成对这些基础协议的修修补补竟然要花费十几年甚至二十年的时间。标准一旦确定得到普遍认可就很难再改变，这就像人的刻板印象一样。

还有，他们是真的很喜欢成立组织啊。

### 人物
- Vint Cerf：“互联网之父”，TCP/IP 的发起者之一。
- Steve Crocker：RFC 备忘录的创始人，ICANN 的前主席，提出了一种提高 DNS 安全性的构想——DNSSEC。
- Jonathan Postel：编写了许多重要的 RFC 规范文档，IANA 的创始人。
  
### 组织/机构
- DARPA：U.S. Defense Advanced Research Projects Agency
- SRI-NIC：Stanford Research Institute International Network Information Center
- ICANN：Internet Corporation for Assigned Names and Number
- IANA：Internet Assigned Numbers Authority
- NFS：National Science Foundation
- NASA：National Aeronautics and Space Administration
- DOE：U.S. Department of Energy
- SAIC：Science Applications International Corporation

[^1]:根域名服务器保存一份 [根域文件](https://www.iana.org/domains/root/files)，文件中包括所有顶级域名权威服务器信息。
[^2]:虽然全世界有成百上千个[根域名服务器的镜像](https://root-servers.org/)，但根域名服务器的 IP 地址是确定的，只有 13 个，后来成为 13 组，每一组根域名服务器共享一个 IP，由于这些 IP 地址用了任播技术，计算机向这 13 个 IP 中的任何一个发起请求时，只会由路由最近的一个根域名服务器响应。