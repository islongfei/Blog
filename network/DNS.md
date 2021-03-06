在日常生活中很容易记得住网站的名称，但是很难记住网站的 IP 地址，因而也需要一个地址簿，就是 `DNS (Domain Name System)` 域名系统服务器。

### DNS 服务器结构  

<img src="https://github.com/islongfei/Blog/blob/master/images/DNS.jpg" width="60%" hegiht="60%"  />
如图所示，DNS服务器是树状的三级层级结构，从上到下依次分为：  

1. 根 DNS 服务器 ：返回顶级域 DNS 服务器的 IP 地址；  
2. 顶级域 DNS 服务器 (.com、.cn ...)：返回权威 DNS 服务器的 IP 地址;
3. 权威 DNS 服务器 ：返回相应主机的 IP 地址。  

### DNS 解析流程  
为了提高 DNS 的解析性能，很多网络都会就近部署 DNS 缓存服务器。就有了以下的 DNS 解析流程：
1. 电脑客户端会发出一个 DNS 请求，问 www.baidu.com 的 IP 是啥啊，并发给本地域名服务器 (`本地 DNS`),
本地 DNS 由网络服务商自动分配，通常就在网络服务商的某个机房。  

2. 本地 DNS 收到来自客户端的请求，如果本地 DNS 服务器上`缓存`了域名与 IP 关系中有 www.baidu.com ， 它直接就返回 IP 地址。
如果没有，本地 DNS 会去问它的根域名服务器。  

3. `根 DNS服务器` 收到来自本地 DNS 的请求，发现后缀是 .com，就会把 .com 的顶级域名服务器的地址给本地 DNS。  

4. 本地 DNS 转向问`顶级域名DNS服务器`：顶级域名服务器是一级域名（.com、.net、 .org 这种）管理的 baidu.com 这些二级域名（中间有两个.），
所以顶级域名服务器说会把负责 xxx.baidu.com 区域的权威 DNS 服务器的地址告诉本地 DNS。

5. 本地 DNS 转向问`权威 DNS 服务器`,权威 DNS 服务器查询后将对应的 IP 地址 X.X.X.X 告诉本地 DNS，客户端和目标 ip 建立连接了。


### 负载均衡  

在域名和 IP 的映射过程中，就有了基于域名做负载均衡优化性能的机会，可以是简单的负载均衡，也可以根据地址和运营商做全局的负载均衡，
分地址和运营商主要是为了返回最优的ip，也就是离用户最近的ip，提高用户访问的速度。  

DNS域名和IP是一对多的关系，一个域名，查找到多个IP地址，根据一定策略选择不同IP，从而实现负载均衡。高可用也与多IP有关，当一个IP有问题时，将其从DNS地址簿移除，本机可以访问其他IP.
