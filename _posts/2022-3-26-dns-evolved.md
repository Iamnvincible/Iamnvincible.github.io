---
layout: post
title: "域名和 DNS 的现代化"
categories: network
tags: dns
permalink: /6/
---


上一篇讲了 DNS 的历史，这一篇讲讲域名本身、DNS 的改进以及更加**现代化**的 DNS 查询方式。

## 域名名称的组成结构
以 `example.com` 这个域名为例，域名以 `.` 将 `example` 和 `com` 分隔为两层，与英语中表示地址的方式类似，最后一层域名 `com` 是顶级域名，`example` 是二级域名。这个域名对应的网址 [example.com](https://example.com) 确实是个可以访问的网站，大概是专门给大家作为域名演示的示例。如果在这个域名的最后加上一个 `.`，这个域名 [example.com.](https://example.com.) 还是可以访问的。最后有 `.` 的域名表示方式称为完全限定域名（Full Qualified Domain Name，FQDN），省略 `.` 的域名表示方式称为部分限定域名（Partially Qualified Domain Name，PQDN）。由于所有域名在最后都会有一个点，所以这个点在大部分情况下被省略。至于为什么会多一个点，可在 DNS 报文中找到原因。DNS 报文在存储域名时并没有 `.` 来分隔域名，而是使用每层域名的**域名长度**和**域名内容**来存储，域名结束时，用 `0` 表示，没有额外的域名内容，即没有上一层域名。就像在 C 语言中表示字符串时，必须有一个 `\0` 来表示字符串结束。在理解时，可以将最后一个点理解为域名的“根”。

### 顶级域名
顶级域名（top-level domain，TLD)，是域名的最后一个词，在 `example.com` 域名中，顶级域名是 `com`。顶级域名由 ICANN 管理，被分为六类[^1]，最新的顶级域名列表可在 [IANA 网站](https://data.iana.org/TLD/tlds-alpha-by-domain.txt) 获得。
- 基础设施顶级域名，只有 `arpa` 这一个，是阿帕网的缩写，是第一个 Internet 顶级域名，最早用于转换阿帕网主机名到 DNS 名称，现用于反向域名解析，例如 `in-addr.arpa` 域名用于从 IPv4 地址找到对应的域名，`ip6.arpa` 用于从 IPv6 地址找到对应的域名 。
- 通用顶级域名（generic top-level domains，gTLD），包括 `com`、`net`、`org` 等互联网早期创立的域名以及新千年后 ICANN 批准的域名，到 2022 年已经有 1245 个顶级域名，可细分为下面三类。
  - 一般通用顶级域名，通用顶级域名一般没有注册限制，但也有一些需要满足特定条件才能申请。
  - 地域类域名，用于特定地域、民族或文化社群，例如 `africa`、`tokyo`。
  - 商标类域名，用于企业或商标，例如 `bing` 域名是微软必应搜索引擎商标，`apple` 域名是苹果公司商标，这些域名可能现在并没有实际子域名可用，但为了保护商标，商标持有者会向 ICANN 申请这些域名[^2]。
- 通用受限域名，只有 3 个。`biz` 用于商业；`name` 用于个人用户，但实际没有限制；`pro` 用于有专业资质个人或组织，但现在也没有限制。
- 赞助类顶级域名，一些域名现由特定机构赞助，共 14 个，这些域名注册需要得到赞助机构批准，例如 `gov` 域名在互联网出现早期就有，但现用于美国政府部门，注册需要美国政府许可；`edu` 域名也出现在互联网早期，现用于美国高等教育机构，注册需要美国教育机构 [Educause](https://www.educause.edu/) 批准。
- 国家和地区顶级域名（Country code top-level domain，ccTLD），以两个字母表示全世界不同国家或地区的顶级域名，共 316 个。国家或地区代码由 [ISO 3166-1 alpha-2](https://www.iso.org/iso-3166-country-codes.html)[^3] 规定，例如 `cn` 代表中国，`hk` 代表香港特别行政区。
  - 其中还包括国际化域名，即以不同语言字符表示的国家或地区域名，例如`中国`、`香港`。
- 测试类顶级域名，共 11 个，用于测试国际化域名，分别是 11 种语言表示的“测试”。

## DNS 服务器
一般情况下，当设备接入互联网时，网络运营商或路由器将会为设备分配 DNS 服务器（或者称为解析器），设备就会从这些 DNS 服务器获取 DNS 查询结果。如果不想使用自动分配到的 DNS 服务器，可以其他 DNS 服务器来替换默认分配到的 DNS 服务器。自动分配的 DNS 服务器和能在网上找到的公共 DNS 服务器可以被称为**递归域名服务器（Recursive name server）**，与之相对的是**权威域名服务器（Authoritative name server）**，两种域名服务器名称的区别与域名服务器获得域名查询结果的方式有关。

### 权威域名服务器
权威域名服务器是实际保存域名记录的服务器，所有 DNS 查询最终都由权威域名服务器来返回结果。举例来说，如果从域名交易商 [namecheap](https://www.namecheap.com) 处购买了域名 `ttmy5.com`，那么这个域名的记录将由 namecheap 提供的域名服务器（nameserver） `dns1.registrar-servers.com` 和 `dns2.registrar-servers.com` 持有，这两个域名服务器就是 `ttmy5.com` 的**权威域名服务器**。域名服务器可以自定义，也就是可以将域名记录转移至其他域名服务器，可以是其他服务器，也可以自行搭建。为使用 [Cloudflare](https://www.cloudflare.com) 提供的服务，自定义域名服务器为 `chelsea.ns.cloudflare.com` 和 `jake.ns.cloudflare.com`。修改后，`ttmy5.com` 域名的**权威域名服务器**就是 `chelsea.ns.cloudflare.com` 和 `jake.ns.cloudflare.com`。如果一个 DNS 查询是直接从查询域名记录所在的权威域名服务器获得的，那么这个查询响应被称为**权威应答**。

### 递归域名服务器
递归域名服务器不保存域名记录，递归域名服务器收到查询请求时，如果没有这个域名的缓存记录，它会从根域名服务器开始，递归地向多个域名服务器查询，直到在域名记录所在权威域名服务器获得查询结果或获得一个“域名不存在”等错误结果后向查询者返回结果。由于查询方获得的结果不是直接来自权威域名服务器，对查询方来说，这样的查询响应被称为**非权威应答**。

### 缓存域名服务器
缓存域名服务器会将每次 DNS 查询的结果保存一段时间以提高查询效率，将 DNS 查询记录缓存下来可以减少网络通信次数，降低权威域名服务器（特别是根域名服务器）的查询压力。缓存域名服务器通常也是一个递归域名服务器，对于一个域名服务器来说，并不一定要实现递归域名服务器的查询过程，向其他缓存或递归域名服务器发起查询并缓存结果已经足够了。

### 根域名服务器
根域名服务器是域名系统的核心，所有递归查询都需要首先向根域名服务器发起查询，由根域名服务器告知顶级域名服务器的 IP 地址。根域名服务器中存有[根区文件](https://www.internic.net/domain/root.zone)，根区文件记录了所有顶级域名所在的权威域名服务器的域名和 IP 地址，例如有关 `com.` 域名的部分记录如下。
```
com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
```
其中每一行都是一条 DNS 记录：第一列是域名；第二列是这条记录的缓存秒数，对于顶级域名来说，一般记录较为固定，缓存时间会是一个比较大的整数，172800 表示缓存时间为两天；第三列为 DNS 类，一般都是 IN，代表 Internet；第四列表示这条 DNS 记录的类型，NS 表示记录是一个权威域名服务器，A 表示记录是一个 IPv4 地址，AAAA 则是 IPv6 地址。
由于顶级域名服务器地址也是一个域名，为避免重复查询，根区文件还会给出这些域名服务器的 IPv4 和 IPv6 地址。
```
a.gtld-servers.net.	172800	IN	A	192.5.6.30
a.gtld-servers.net.	172800	IN	AAAA	2001:503:a83e:0:0:0:2:30
......
m.gtld-servers.net.	172800	IN	A	192.55.83.30
m.gtld-servers.net.	172800	IN	AAAA	2001:501:b1f9:0:0:0:0:30
```
DNS 在设计时就考虑了单点故障问题，根域名服务器共配置了 a 至 m 共 13 组，13 是 早期 DNS 数据包能够容纳服务器数量的最大值。这 13 组根域名服务器由 12 个组织运营，为保证 13 组根域名服务器在全世界范围内的可用性，根域名服务器在全世界部署了任播节点。域名服务器使用任播可以让一组服务器共享一个 IP 地址，在查询时网络负责将消息传递到最近的节点。所有根域名服务器的信息可以[在此](https://root-servers.org/)查询，截至 2022 年 3 月，共有 1524 个根域名服务器节点运行在全世界不同城市。

### 顶级域名服务器
顶级域名服务器存有同一个顶级域名下的所有域名信息，在收到域名查询时，能够给出某个域名所在的权威域名服务器。与根域名服务器一样，一般顶级域名服务器也有多组，并使用任播在全世界部署多个节点保证其可用性。

## DNS 查询过程
从在浏览器输入网址后，浏览器将开始一次 DNS 解析。以访问 `example.com.` 为例，一次理想的解析流程如下，实际情况可能更为复杂。
1. 如果浏览器或操作系统之前已经查询过这个网址所对应的域名，并且没有超过域名的缓存时间，那么浏览器将立即获得结果，无需后续步骤。
2. 向系统或浏览器指定的 DNS 服务器（解析器）发起 DNS 查询。
3. 如果指定的 DNS 服务器已经有所查域名的缓存，那么 DNS 服务器立即返回这个缓存的结果，无需后续步骤。
4. 指定的 DNS 服务器一般是缓存域名服务器，缓存域名服务器可能会向其他缓存域名服务器发起查询，中间任何一个缓存服务器有缓存记录都可以立即返回。
5. 当查询发送到了一个递归域名服务器上且没有可用缓存时，真正的域名查询过程开始。
6. 首先查询根域名服务器的地址。
   > 一般来说，根域名服务器的 IP 地址很少发生改变[^4]，递归域名服务器上通常都会有一个[根域名服务器缓存](https://www.internic.net/domain/named.root)文件，这个文件记录了所有根域名服务器的信息，正常情况下直接读取缓存文件中的根域名服务器地址即可。
7. 向其中一个根域名服务器（比如 `b.root-servers.net.`）发送要查询的域名 `example.com.`。
8. 根域名服务器 `b.root-servers.net.` 将返回 13 个 `com.` 顶级域名服务器地址。
9. 选择一个 `com.` 顶级域名服务器（比如 `d.gtld-servers.net`），向其发起域名 `example.com.` 的查询。
10. `d.gtld-servers.net` 返回 `example.com.` 的 `example.` 二级域名所在的权威域名服务器地址，有两个分别是 `a.iana-servers.net.` 和 `b.iana-servers.net.`。
11. 选择一个 `example.com.` 域名的二级域名所在的权威服务器（比如 `b.iana-servers.net`），向其发起域名 `example.com` 的查询。
12. `b.iana-servers.net` 是域名 `example.com.` 所在的权威域名服务器，实际记录了与 `example.com.` 域名相关的记录，它查询记录后返回 `example.com.` 的实际 IP 地址 `93.184.216.34`。
13. 第 5 步中的递归域名服务器向查询方返回第 12 步获得的响应。

在 Linux 系统中，可以使用 `dig` 在命令行查询 DNS，`dig` 支持很多选项，`+trace` 选项可以从根域名查询开始追踪一次 DNS 查询，例如 `dig example.com +trace`，其输出内容较多，此处不便列出。不过 `+trace` 发起的查询和真正的递归查询并不一样，只能当作是一种模拟。

## DNS 的现代化
从 DNS 开始应用至今，大部分 DNS 查询和响应都是通过 UDP 协议的 53 端口来通信，查询方和响应方的数据都是明文，在网络传输中的任何一个环节都可以获得查询和响应内容。
明文传输意味着传输线路上任何设备都能看到 DNS 查询内容，这点可能导致隐私泄露，但更重要的是传输过程中可能获得伪造的 DNS 响应。例如，攻击者会为了让查询方无法访问特定域名而伪造这些域名的 DNS 记录。攻击者也可以通过伪造 DNS 记录而将查询方的访问劫持到某个恶意网站，例如，如果访问 `example.com` 时获得的 DNS 响应 IP 不是真正 `example.com` DNS 记录中的 IP 地址而是某个恶意网站的 IP，查询方将进入这个恶意网站而难以察觉。幸运的是，在网站使用 HTTPS 后，即使 DNS 查询获得了错误的 IP，浏览器也会因为证书错误而停止访问该网站。
UDP 是无连接的传输层协议，在 DNS 查询发出后，攻击者会伪造响应数据，在真正的响应到达前将结果发送回查询方。由于 UDP 无连接的性质，查询方不能辨别响应方是否是真正的 DNS 服务器而采用先到达的响应作为结果。如果查询方是缓存域名服务器，将造成 DNS 缓存污染，所有从该缓存域名服务器获得查询结果的查询方都将收到影响。

为解决 DNS 存在的问题，先后提出了多种解决方案，这里介绍几种常见的方案。

### 使用 TCP 的 DNS
[RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035) 规定了 DNS 服务器需要同时监听 TCP 和 UDP 的 53 端口。UDP 不需要建立连接，传输速度较快，但不能保证可靠性，传输的 DNS 部分数据的长度也不能超过 512 字节；TCP 传输需要建立连接，传输更加可靠，传输的数据长度也更长，一般用于一组 DNS 服务器之中的备份服务器和主服务器的数据同步。在 DNS 消息长度大于 512 字节时，会从 UDP 改为使用 TCP 来传递数据。
使用 TCP 方式查询似乎更加可靠，但由于传输内容仍是明文，仍然能被攻击者看到查询的域名。与 UDP 查询方式的攻击方式不同，由于 TCP 基于连接，攻击者可以发送一个 TCP 连接重置数据包以中断查询方和 DNS 服务器的连接，这样查询方就不能收到 DNS 的查询结果，也就不能正常访问特定域名。
使用 `dig` 时可以加上 `+tcp` 选项来指定使用 TCP 来发起 DNS 查询。不过，一些公共 DNS 服务器似乎并没有开放 TCP 53 端口，操作系统也没有提供设置 DNS 使用 TCP 的选项，这种方式更多用于测试。

### DNSSEC
在 DNS 设计时并没有考虑安全问题，查询方在收到响应时，没有对结果验证的步骤，也就是在收到一个 DNS 响应时无条件信任。在 Internet 发展的早期，网络环境没有现在这么复杂，接入的设备也没有那么多，DNS 的使用似乎没有问题，但随着接入设备的增加，网络规模逐渐扩大，DNS 的潜在问题渐渐暴露。域名系统安全扩展（Domain Name System Security Extensions，DNSSEC）为 DNS 打上了补丁，DNSSEC 使用了数字签名技术来验证 DNS 响应内容的正确性。
DNSSEC 并不是对 DNS 数据本身的加密，而是 DNS 记录的拥有者对 DNS 数据的签名，所以 DNS 查询和响应内容仍然是明文，不过现在可以验证 DNS 结果的正确性，查询方不会再被篡改的 DNS 响应记录欺骗。
DNSSEC 为 DNS 添加了两项重要功能，数据来源验证和数据完整性保护，可以确保数据来自权威域名服务器以及数据在传输过程中不被修改。递归域名服务器在查询域名时会同时查找域名的公钥，通过公钥验证 DNS 数据的真实性，如果验证有效，域名服务器将 DNS 数据返回给查询方，否则丢弃数据并向查询方返回错误。
域名的每个层级都会发布公钥，而每个层级的公钥由上一级域名的私钥签名（根区除外）。例如，域名 `icann.org` 的公钥由 `org` 域来签名。根区是所有域名信任的起点，在信任根区公钥之后，可以进而信任由根区私钥签名过得顶级域名的公钥，例如 `org` 域。在信任 `org` 域的公钥后，可以进一步信任由 `org` 域私钥签名的子域名，例如 `icann.org`。更进一步地，每一个域名可以为其次级域名签名，从而完成整个域名的签名流程。不过在实际的 DNSSEC 认证过程中，上级域名并不直接对次级域名签名，签名机制更为复杂。
#### DNSSEC 实现
DNS 记录有多种类型，上文提到了 A 记录、AAAA 记录、NS 记录，常见的还有 MX 记录——用于指向电子邮件服务器，CNAME 记录——用于指向另一个域名。
##### DNSSEC 增加的记录类型
- RRsig 记录，资源记录集的数字签名。
- DNSKEY 记录，用于存放 DNSSEC 机制中用到的公钥内容。
- DS 记录，DNSKEY 的摘要。
- NSEC 和 NSEC3 记录，用于**明确否认**域名记录不存在。
- CDNSKEY 和 CDS 记录，用于向上级域请求更新 DS 记录。
##### 实现步骤
- 每一种 DNS 记录类型会有多条记录，每一种记录类型的集合称为资源记录集（Resourc Record Set，RRset）。例如，所有的 A 记录组成一个 A 记录的 RRset。
- 生成域签名密钥对（Zone-Signing Keys，ZSK）。
  - ZSK 中的私钥为每一个 RRset 的内容签名，得到这个 RRset 的资源记录签名（RRsig）。在查询一个域名的 A 记录时，结果中不仅会有这个域名的几个 IPv4 地址，还会附有一个 RRsig 类型的记录，这个记录就是对查到的所有 A 记录的数字签名。
  - 本域内所有 RRset 的签名都是在生成密钥对时计算得到的，而不会在每次查询时计算，这样可以减轻服务器的压力。
  - ZSK 中的公钥将作为 DNSKEY 记录。
- 生成另一组密钥对——密钥签名密钥对（Key-Signing Keys，KSK）。
  - KSK 中的公钥作为 DNSKEY 记录。
  - KSK 中的私钥对 DNSKEY 记录的 RRset 签名，得到 RRsig。查询方在验证记录时需要查询 DNSKEY 记录，结果中不仅有两个 DNSKEY 记录，还会同时会获得一条 RRsig 记录，这是 DNSKEY 记录的数字签名。
- 向上层域提交委派签名（Delegation Signer，DS）记录。
  - DS 是 KSK 中公钥的摘要。
  - 每一层域的 DS 都交给上一层域记录。
  - 每一层域名 KSK 公钥的真实性由上一层域名的 DS 记录保证，这就形成了**信任链**。
##### 验证示例
以查询、验证 `example.com` 为例。
1. 查询 `example.com` 的 A 记录，得到域名 IPv4 地址为 `93.184.216.34`，以及一条 RRsig 记录。
2. 为验证记录的真实性，查询 `example.com` 的 DNSKEY 记录，获得 ZSK 公钥、KSK 公钥以及 DNSKEY 记录的 RRsig。
3. 为验证 ZSK 公钥的真实性，用 KSK 公钥解密第 2 步的 RRsig，以确定 ZSK 公钥有效。
4. 用 ZSK 公钥解密第一步的 RRsig 记录，以确定 A 记录内容有效。
5. 为验证 KSK 公钥的真实性，向 `.com` 域查询 `example.com` 的 DS 记录。
6. 计算 KSK 公钥的摘要值与 `.com` 域提供的 DS 记录比较，以确定 KSK 公钥有效。
7. 查询 `.com` 域的 DNSKEY 记录和 RRsig。
8. 向根域查询 `.com` 域的 DS 记录。
9. 比较第 7 步获得的 KSK 公钥摘要值和第 8 步获得的 DS 记录，以确定 `.com` 域名的 KSK 有效。
10. 查询根域名的 DNSKEY 记录和 RRsig。
11. 用根域名提供的 KSK 公钥解密第 10 步获得的 RRsig，以确定 KSK 公钥有效。

这个例子可以帮助理解，但验证过程和步骤并不完整，而且实际 DNSSEC 的验证是从根域名开始的。
#### 根域 DNSKEY 记录如何签名？
在上面的验证过程中到 11 步结束，这是因为根域没有可用的上级域用于提供 DS 记录供验证，所以根域的 RRsig 产生过程至关重要。这个过程充满了人文气息，RRsig 是通过[根区签名仪式](https://www.cloudflare.com/zh-cn/dns/dnssec/root-signing-ceremony/)产生的！仪式通常一年举行四次，受新冠疫情影响，2020 和 2021 年两年只举行了四次，仪式过程向全世界直播，可以在 [IANA 的频道](https://www.youtube.com/channel/UChND9hEeJQjtLDFZ-m8U47A)观看历次仪式的录像。
参与根区签名仪式的人员从 ICANN 推选的值得信赖的社区代表中产生，仪式过程十分严格，需要 3-4 小时才能完成。严密的步骤使我们相信根区签名的有效性，进而信任整个 DNSSEC 机制。

> 注意到，在实现步骤中生成了两对密钥对，这是因为 KSK 的更新不仅需要在本域内更新签名，还需要在上级域更新 DS 记录（CDNSKEY 和 CDS 记录用于完成更新任务），这个过程较为复杂而且需要等待原记录过期，如果出错可能导致整个域无法访问[^5]。使用两套密钥时，可以在需要更新 ZSK 时，重新对本域内所有 RRset 的签名即可，不需要与上级域交互。
> 除了要对存在的域名进行验证，DNSSEC 还能对不存在的域名做出**明确否认**，这是 NSEC 和 NSEC3 记录的功能。

#### DNSSEC 的使用现状
2010 年，根域名完成了 DNSSEC 的签名，但在全世界范围内部署这项对基础协议的升级还需要一些时间。启用 DNSSEC 需要权威域名服务器的支持，而且需要域名拥有者手动启用，这的确会遇到一些阻力。举例来说，一些知名公共 DNS，例如 114、DNS Pod、AliDNS 都还没有启用 DNSSEC 功能，运营商提供的 DNS 就更没有这项功能了。
此外，即便选择的 DNS 服务器支持了 DNSSEC，本地要使用 DNSSEC 方式查询仍需要安装特定软件。而且并非所有域名都启用 DNSSEC，客户端使用 DNSSEC 的效果有限。
虽然 DNSSEC 能够 DNS 查询的正确性，但由于其传输的数据依然是明文的，不能保证查询方和 DNS 服务器之间的通信安全和保密性，攻击者仍可以干扰查询的过程。在这样的情况下，使用 DNS over TLS、DNS over HTTPS 等经过加密的 DNS 查询方案更为安全。

### DNS over TLS
为解决 DNS 明文传输带来的风险，DNS over TLS 或称 DoT 使用传输层安全协议（TLS）来加密 DNS 查询和响应内容。传输层安全协议和 HTTPS 网站所使用的加密通信协议相同，只不过其承载的是 DNS 数据内容。有了这层加密，DNS 的查询和响应内容将不会被攻击者窥探或伪造。
[RFC 7858](https://datatracker.ietf.org/doc/html/rfc7858) 规定 DoT 服务器监听 TCP 853 端口，除非由其他约定。在约定端口不能传输任何明文的 DNS 数据以保证传输的秘密性。查询方和 DNS 服务器在确定的 DoT 端口建立 TCP 连接后，开始 TLS 握手，最终传递加密的 DNS 消息。
传统指定 DNS 服务器时需要指定目标 DNS 服务器的 IP 地址，而使用 DoT 时，一般需要指定 DNS 服务器的域名，例如 `dnsserver.example.net`。既然用到了域名，就需要一次传统的 DNS 查询，以便获得对应的 IP 地址，也可以通过别的办法避免这个初始 DNS 查询。使用域名还有一个作用，即验证域名对应的数字证书，可以进一步提高通信的安全性。
DoT 使用了特定的传输端口，这对于需要识别 DNS 查询的网络中可能会有帮助，但如果要进一步提高私密性，避免通过端口特征识别可能的意图，可以使用 DNS over HTTPS。

### DNS over HTTPS
使用专用于 DNS 查询的端口可能会被攻击者阻断，DNS over HTTPS 或称 DoH 可以将 DNS 查询和响应内容隐藏于常规 HTTPS 流量中。HTTPS 协议也是基于 TLS 来加密，通常监听 TCP 443 端口，常用于加密数据传输，能提供较好的私密性。DoH 将 DNS 查询隐藏于普通的加密网页数据中，除非攻击者完全阻断 HTTPS 流量（这就意味这不能访问整个网站，包括 DoH 和常规的网页内容），否则很难将 DNS 数据从流量巨大的 HTTPS 中分离出来专门阻断。
由于 DoH 基于 HTTPS 实现，使用时需要指定一个提供 DoH 的 URI，[RFC 8484](https://datatracker.ietf.org/doc/html/rfc8484) 给出的模板 URI 为 `https://dnsserver.example.net/dns-query{?dns}`，也就是提供 DoH 服务的服务器域名为 `dnsserver.example.net`，而在 `dns-query` 这个位置下提供的是 DoH 服务，其他位置与 DoH 无关。一个服务器可以提供很多服务，其中 `dns-query` 这个位置提供的是 DoH 服务。当然也可以用其他位置来提供 DoH 服务，这是使用 HTTPS 的优点——可以比较容易地为一个 HTTPS 服务器增加对外的 DoH 服务而不影响原有服务。
作为查询方使用 DoH 时，在需要填写 DoH 地址的地方填写 `https://dnsserver.example.net/dns-query` 即可，模板中后面参数由 DNS 查询软件自动填写。

### 更多 DNS 查询方式
- DNS over QUIC
  QUIC 是一种新设计的传输层协议，相比 TCP 有更好的速度和稳定性。DNS over QUIC 基于这种新的协议传输，同样能保证传输安全和隐私，但由于协议较新，只有 AdGuard 等少数服务商提供这样的 DNS 查询方式。
- DNSCrypt
  一种还没有以 RFC 征求意见稿方式提交给 IETF 的 DNS 查询方案，不过已经有多个服务端和客户端的实现，许多知名 DNS 服务商已经提供这种 DNS 查询服务。DNSCrypt 需要先以传统 DNS 查询方式获得服务端的证书，再进行加密通信。
- DNSCurve
  - 一种基于椭圆曲线加密算法的 DNS 查询方式，但很少有 DNS 服务商支持。

### EDNS
为了在 DNS 协议中增加更多功能，[RFC 2671](https://datatracker.ietf.org/doc/html/rfc2671) 提出了 DNS 扩展机制（Extension Mechanisms for DNS，EDNS）。DNS 是一个广泛使用的协议，为了保证执行原有协议的服务器仍能正常处理，EDNS 引入了一种伪资源记录（OPT）添加在 DNS 消息的附加记录中。
#### 消息长度
启用 EDNS 后，查询方在消息中加入 OPT 记录，可以告诉 DNS 服务器支持的 DNS 消息长度。这个选项可以使通过 UDP 发送的 DNS 消息长度超过 512 字节。
#### EDNS Client Subnet（ECS）
另一个 EDNS 选项，可以发送查询方的网段到权威域名服务器。这个功能可以在查询方使用地理位置上相距较远的 DNS 服务器时，将自己所在 IP 段发送给权威域名服务器，权威域名服务器可以依据查询方 IP 段返回一个离查询方较近的目标域名的 IP 地址，使查询方连接时有更快的访问体验。
网络应用服务商为了提高各地用户的访问速度，会在不同地方使用 CDN（内容分发网络）服务器缓存资源，在用户访问时会分配离用户最近的服务器。在没有 ECS 功能时，服务商只能根据查询方指定的 DNS 服务器 IP 地址来确定 CDN 服务器，如果查询方使用的 DNS 服务器离其地理距离很远——例如，A 国的用户使用了架设在 B国 DNS 服务器，那么服务商会分配 B 国的 CDN 服务器给 A 国用户，用户访问资源就需要更长的时间。而如果使用了 ECS 功能，无论查询方使用哪里的 DNS 服务器，服务商都能为查询方分配最近的 CDN 服务器。

## 现代化查询 DNS 的方法
为了安全地获得正确的 DNS 响应，在互联网上通过传统 UDP 53 端口查询的方式似乎不再是一个好的选择。使用 DoH 和 DoT 可以和 DNS 服务器加密通信，这样可以避免与 DNS 服务器通信过程中的攻击。通信的过程安全了，保证结果正确的责任就在选择的 DNS 服务器上，因为即使查询方和 DNS 服务器的通信经过加密，如果 DNS 服务器本身获得的 DNS 记录不正确，查询方仍会获得错误的结果。AdGuard 提供了一份[世界范围内的公共 DNS 列表](https://kb.adguard.com/en/general/dns-providers)，可以从中挑选访问较为稳定的使用。
由于一些众所周知的问题，通过 DoH 查询可能会比 DoT 查询可用性更高，但通过 DoH 查询也可能在一些时候难以使用，如果只是为了加密和 DNS 服务器之间的通信，可以选择 [alidns](https://alidns.com)、[DNSPod](https://docs.dnspod.cn/public-dns/dot-doh/) 等提供的 DoH/DoT。如果希望获得更快、更准确的结果，可以使用提供[非标准 UDP 端口查询的公共 DNS 服务器](https://ilpl.me/2018/08/17/public-DNS-5353-list/)，在不介意明文传输的情况下能获得很好的体验（至少目前如此）。

### 设置方法
#### 浏览器设置
Firefox 和基于 Chromium 的浏览器已经可以在网络设置中指定 DoH 方式查询。不过，可能由于浏览器还没有使用 ECS，设置一些国外 DoH 服务商时，网页访问速度会明显变慢。
#### Android/iOS
从 Android 9 起，可以在设置中搜索“私人 DNS”或“加密 DNS”来设置全局的 DoT。如果要使用 DoH，需要下载一些 APP 来支持，例如 [Intra](https://play.google.com/store/apps/details?id=app.intra)、[1.1.1.1](https://play.google.com/store/apps/details?id=com.cloudflare.onedotonedotonedotone)、[personalDNSfilter](https://www.zenz-solutions.de/personaldnsfilter-wp/)。
从 iOS 14 起，可以下载特定的描述文件来支持 DoH/DoT。iOS 14.1 起似乎可以在设置里添加 DoH/DoT 服务器地址了？我也不知道，我也没用过😁。
#### Windows/macOS/Linux
从 Windows 11 起，可以在网络设置中设置 DoH。macOS，😁。Linux 需要安装支持 DoH/Dot 的本地 DNS 解析器。
#### 进一步地
设置国外 DNS 时，会出现访问变慢地情况，这就需要为不同域名指定不同的 DNS 或使用 ECS 来加快访问，在客户端简单设置不能达到这种效果，除非在 DNS 服务端处理这些问题，可以 DIY 一个自己的 DNS 或者选择一个会处理这些问题的可靠 DNS。
使用自定义的 DNS 还可以用来屏蔽一些广告、追踪和恶意网站，[AdGudard](https://adguard.com/zh_cn/welcome.html) app 为不同平台提供了解决方案，不仅能完成广告屏蔽，还能设定加密 DNS。

### 命令行查询 DNS 工具
- [dig](https://www.isc.org/download/)，dig 是 BIND 中的命令行 DNS 查询工具，使用非常广泛。从版本 9.18 起，dig 支持了 DoH 和 DoT。
- [dnslookup](https://github.com/ameshkov/dnslookup)，支持 UDP、DoH、DoT、DoQ、DNSCrypt。
- [doh-cli](https://github.com/libreops/doh-cli)，Python 编写的 DoH 查询工具。
- [dog](https://dns.lookup.dog)，和 dig 名字很像，支持 UDP、TCP、DoT、DoH。

### DIY DNS 可能会有用的程序、资源
- [dnsmasq](https://dnsmasq.org/)
- [overtune](https://github.com/shawn1m/overture)
- [mosdns](https://github.com/IrineSistiana/mosdns)
- [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome)
- [anti-AD](https://github.com/privacy-protection-tools/anti-AD)

### 本文参考到的链接
- [Domain name resolution Arch doc](https://wiki.archlinux.org/title/Domain_name_resolution)
- [DNS的历史和原理](https://yangwang.hk/?p=852)
- [DNS 原理入门](https://www.ruanyifeng.com/blog/2016/06/dns.html)
- [根域名的知识](https://www.ruanyifeng.com/blog/2018/05/root-domain.html)
- [Name server](https://en.wikipedia.org/wiki/Name_server)
- [根服务器系统概述](https://www.icann.org/en/system/files/files/octo-010-06may20-en.pdf)
- [根相关文件](https://www.iana.org/domains/root/files)
- [DNSSEC 如何运作](https://www.cloudflare.com/zh-cn/dns/dnssec/how-dnssec-works/)
- [DNSSEC - 它是什么？为什么说它很重要？](https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-20-zh)
- [DNSSEC 技术详解](https://init.blog/dnssec/) 
- [Root Zone Operator Information](https://www.iana.org/dnssec)

[^1]: 分类依据来自[根区数据库](https://www.iana.org/domains/root/db)。
[^2]: [商标类顶级域名列表和申请条件](https://newgtlds.icann.org/en/applicants/agb/base-agreement-contracting/specification-13-applications)。
[^3]: ISO 3166 是为国家、地区指定代码的标准，其中 IOS 3166-1 是标准的第一部分，规定了国家和地区的代码。而 ISO 3166-1 alpha-2 指的是以 2 个字母来表示的国家/地区代码，ISO 3166-1 alpha-3 则是用 3 个字母表示的国家/地区代码。
[^4]: 根域名服务器的 IP 地址、域名[有时候](https://www.icann.org/en/system/files/files/rssac-023-04nov16-en.pdf)确实会发生改变，在其中一个不能使用时可以用剩下的来查询并获取更新的根域名服务器列表，域名系统的设计能够很好应对这样的变化。
[^5]: 就在 2022 年 3 月，一次 DNSSEC 的错误配置使得斐济的国家顶级域名 `.fj` 从互联网上消失了一段时间，见[DNSSEC issues take Fiji domains offline](https://blog.cloudflare.com/dnssec-issues-fiji/)。