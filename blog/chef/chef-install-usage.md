<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-06-12
title: Chef 的安装与使用
tags: Chef
images: https://os4u.info/blog/img/sun.png
category: Chef
status: publish
summary: Chef 是一款自动化服务器配置管理工具,理论上可以对服务器做任何配置,包括系统管理、安装软件等,近来已被越来越多地应用到云环境的自动化部署上。运维必备自动化神器之一，所以还是要学习下。
-->


# Chef 的安装与使用


![](https://www.os4u.info/blog/chef/images/start_chef.svg)

Chef是一款自动化服务器配置管理工具，可以对所管理的对象实行自动化配置，如系统管理，安装软件等。Chef由三大组件组成：Chef Server、Chef Workstation和Chef Node。

Chef Server是核心服务器，维护了一套配置脚本（Cookbook），与每个被管节点（Chef Node）交互并给出配置指令。

Chef Workstation提供了我们与Chef Server交互的接口：在Workstation上创建定义Cookbook，并将Cookbook上传到Chef Server上以保证被管机器能从Chef Server上取得最新的配置指令。

Chef Node是安装chef-client并注册的被管理节点，可以是物理机或者虚拟机或者其他对象。Chef Node每次运行 chef-client时都会从Chef Server端取得最新的配置指令（Cookbook）并按照指令配置自己。

一套Chef环境: 包含一个Chef Server，至少一个 Chef Workstation，以及一到多个 Chef Node。

* * *

## Chef环境的安装

Chef环境的安装步骤一般是：先安装Chef Server，然后配置Chef Workstation, 最后根据需要在客户端机器上安装 Chef Client并将其注册成Chef Node。

Chef Server和Chef Workstation可以配在一台机器上，也可以分开配置。由Chef Server、Chef Workstation和多个Chef Node组成整个Chef环境。

在Chef的官网上有详细的Chef安装步骤说明，官网提供的是在有外部网络环境的前提下利用网络自动下载和安装软件。

本文将根据实践提供一个无外部网络环境下的Chef环境安装过程。

### 准备工作

由于服务器是无外部网络环境的，事先将所需的软件包下载到本地准备好，本次部署使用的是内部镜像服务器。

[制作自定义rpm软件包](https://os4u.info/blog/linux/centos-rpm-package.html)

[yum server on centos](https://os4u.info/blog/linux/centos-yum-server.html)

查询chef server安装包

```
# yum search chef-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
============== N/S matched: chef-server ==============
chef-server-core.x86_64 : The full stack of chef-server

  Name and summary matches only, use "search all" for everything.
``` 
查询Chef Workstation安装包

```
# yum search chefdk
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
============== N/S matched: chefdk ==============
chefdk.x86_64 : The full stack of chefdk

  Name and summary matches only, use "search all" for everything.
```
查询Chef Client安装包

```
# yum search chef
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
============== N/S matched: chef ==============
chef.x86_64 : The full stack of chef

  Name and summary matches only, use "search all" for everything.
```

### 安装Chef Server包

```
# yum install chef-server-core
```

配置 Chef Server 12.x： (确保防火墙已关闭)

```
# chef-server-ctl reconfigure
```
**【注】** 此命令会创建 Chef Server12.x 的所有必需组件，包括 Erchef、RabbitMQ,、PostgreSQL 等。

验证 Chef Server 12.x 是否安装成功：

可通过两种方式验证：

```
a. 在 Chef Server 上运行"# chef-server-ctl test"命令.
	此命令会运行 chef-pedant 的测试组件并报告所有组件正常工作，安装正确。

b. 直接在浏览器中打开 Chef Server 的页面： https://Chef_Server_IP。
	若能出现登录界面，说明 Chef Server 已经正确启动。
```

### 安装Chef Workstation

登陆到 Chef Workstation 服务器上，按照如下步骤配置Chef Workstation：
安装 Chef Client 安装包：

```
# yum install chefdk.x86_64
```
验证 chef-client 已经安装成功：

```
# chef-client -v
Chef: 12.19.36
```

确定一个作为 Chef Repository 的目录，如创建/home/chef 目录（后面将以此为例）。

将 Chef Repository 包（chef-repo-master.zip）解压并拷贝到/home/chef 目录，重命名为 chef-repo。
在/home/chef 下创建.chef 目录。

将 Chef Server 上的 admin.pem 和 chef-validator.pem 文件(位于/etc/chef-server) 拷贝到 Chef Repository 的.chef 目录中。

运行"knife configure --initial" 命令配置 Chef Workstation，示例如下：

* 清单 1.配置Chef Workstation
	
	```
	# knife configure --initial
	Where should I put the config file? (/root/.chef/knife.rb) /home/chef/chef-repo/.chef/knife.rb
	Please enter the chef server URL: [https://localhost:443] https://Chef_Server_IP:443
	Please enter a name for the new user: [root]
	Please enter the existing admin name: [admin]
	Please enter the location of the existing admin's private key: [/etc/chef-server/admin.pem] 
	/home/chef/chef-repo/.chef/admin.pem
	Please enter the validation clientname: [chef-validator]
	Please enter the location of the validation key: [/etc/chef-server/chef-validator.pem] 
	/home/chef/chef-repo/.chef/chef-validator.pem
	Please enter the path to a chef repository (or leave blank): /home/chef/chef-repo
	Creating initial API user...
	Please enter a password for the new user:
	Created user[root]
	Configuration file written to /home/chef/chef-repo/.chef/knife.rb
	# ls
	admin.pem  chef-validator.pem  knife.rb  root.pem
	```

添加环境变量：

```
# echo 'export PATH="/opt/chef/embedded/bin:$PATH"' >> ~/.bash_profile && source ~/.bash_profile
```

验证 Chef Workstation 是否配置成功：

运行"knife client list"和"knife user list"进行验证，如清单 2 所示。

* 清单 2. 验证 Chef Workstation

	```
	# cd /home/chef/chef-repo
	# knife client list
	chef-validator
	chef-webui
	# knife user list
	admin
	root
	```

### 安装 Chef Client
将 Chef Client 安装包上传到目标机器上，登陆到此机器上，按照如下步骤配置 Chef Client：
安装 Chef Client 安装包：

```
# yum install chef.x86_64
```
验证 chef-client 已经安装成功：

```
# chef-client -v
Chef: 13.1.31
```

确保Chef Client机器与Server的时钟是同步的(相差少于15分钟)


### 将安装了Chef Client的此机器注册成一个Chef Node。

在 Chef Workstation 上运行 bootstrap 命令：

```
# knife bootstrap Chef_Client_IP -x username -P password
```

**【注】** bootstrap 命令会检查客户端有没有安装 chef-client 软件，如果没有安装，会从网络上直接下载安装包去安装然后注册；如果已经安装了，则会直接将这个客户端注册成一个 Chef Node。可以接着在 workstation 上执行 node list 命令查看是否多了一个 node：

```
# knife node list
```

## Chef的使用
Chef 环境安装完成以后，我们来看看如何使用这套环境来进行配置管理。Chef的配置过程是：

* 1.在Workstation上定义各个Chef Client应该如何配置自己，然后将这些信息上传到Server端。
* 2.每个Chef Client连到Server查看如何配置自己，然后进行自我配置。

在 Workstation上使用 Cookbook 来定义配置方法。Cookbook使用Ruby脚本定义对ChefClient的各种操作，具体Cookbook的写法本文不做叙述。

一旦Cookbook写好之后，就可以重复使用，可以对多个Chef Client进行批量配置。一般从创建Cookbook到使用Cookbook会包括以下几个过程:

a) 在Workstation上创建Cookbook

使用knife命令可以快速创建一个Cookbook，如：

```
# knife cookbook create db2
```

b) 编辑 Cookbook

根据实际需要，编辑Cookbook里的Recipe，可以定义各种对服务器的配置操作，如系统管理，安装软件等。

c) 同步Cookbook

将在 Workstation 上写的 Cookbook 同步上传到 Server 上，可以通过 upload 命令实现：

```
# knife cookbook upload db2
```
**【注】** 

* 此命令将最新的 db2 Cookbook 上传到 Server 端，这样 Client 端就能从 Server 端得到最新的配置指令。
* 也可以将所有的 Cookbook 都一起上传到 Server 端：

	```
	# knife cookbook upload --all
	```
	
d) 将Cookbook添加到要配置的Node的Run List中，如：

```
# knife node run_list add chef-node2 recipe[db2]
```

***【注】*** 

* 此命令将名为 db2 的 Cookbook 下的默认 Recipe（default.rb）添加到名为 chef-node2 的 Node 的 run_list 中。
* 也可以指定 Cookbook 里的某个特定 Recipe 添加，如：
	
	```
	# knife node run_list add chef-node2 recipe[db2::createdb]
	```
	此命令就将特定的 Recipe(created.rb)添加到 Node 的 Run List 里。

e) 可以通过 knife 的 node show 命令查看某个 Node 的具体信息：

```
# knife node show chef-node2
```

***【注】*** 此命令可以看到 Node 的 Run List。
查看更详细的 Node 信息可以加上-l 参数：

```
# knife node show –l chef-node2
```

### 运行Cookbook

在Chef Node上直接运行`chef-client`命令，Chef Client就会从 Server端下载最新的配置脚本，然后按照配置脚本配置自己（即脚本运行的过程）。

除在Client端直接运行`chef-client`命令，也可以在Workstation上运行`knife ssh`命令来达到同样的效果。

不同的是，在 Client 端运行`chef-client`命令只是对自己一个 Node 进行配置，而在Workstation上运行的`knife ssh`命令可以同时对多个 Client 端进行批量配置。

#### Chef的API调用

在实际使用中，我们经常需要将Chef集成到已有的系统中，这个时候就需要调用Chef的API来完成。Chef本身提供了REST API，可以方便的被调用。

只是有少许特殊功能REST API不能完成（如注册Chef Node），还需要调用Chef的命令行。

本节先介绍Chef的REST API，然后讨论Chef的命令调用。

#### 调用Chef REST API

Chef的REST API提供了对Chef内对象的

* 增删改查操作，如增加、删除一个节点、修改节点属性；
* 查询一个 Cookbook。

具体的每个API可以在Chef官网中找到, 我们主要对调用一个REST API的具体过程做出说明。

a) 首先，调用Chef的REST API之前需要与Chef Server端建立认证。

Chef的认证是基于公私钥的非对称加密机制对用户进行认证。

Chef Server为每个客户端（Workstation，Node 或是其他向 Chef Server发送请求的应用）生成一对独立的公钥和私钥，将私钥返回给客户端而自己持有所有客户端的公钥。

当持有私钥的客户端发送请求时，必须用自己的私钥对请求内容制作数字签名，并随同请求一起发送。Chef Server用该客户端的公钥对请求中的数字签名进行验证，如果成功，则认为请求发送方可以信任。

这里的一个特殊情况是，当客户端第一次向Chef Server发送请求，即请求为自己生成公钥私钥对时，尚未持有自己的私钥。此时客户端必须通过某种方式获取Chef Server的默认私钥，通常命名为 Chef-validator.pem。客户端必须用这一私钥执行上述签名过程，否则将无法建立与Chef Server的信任。

认证过程如下图所示，

![图 1 Chef 认证流程](https://www.ibm.com/developerworks/cn/cloud/library/1407_caomd_chef/image002.png)

<center> 图 1 Chef 认证流程 </center>

在上图的第四步，数字签名的必须包含如清单 3 所示的内容。

* 清单 3.Chef 所要求的数字签名内容

	```
	  Method:HTTP_METHOD
	  Hashed Path:HASHED_PATH
	  X-Ops-Content-Hash:HASHED_BODY
	  X-Ops-Timestamp:TIME
	  X-Ops-UserId:USERID
	```

签名的值被放到一个特定的 HTTP Header 'X-Ops-Authorization-N'里发送。这里N表示这个 header可以出现多次，每次出现时它的值不能超过60个字符。一个典型的带有签名的客户端请求如清单4所示。

* 清单 4.带有数字签名的请求范例
 
	```
	GET /organizations/organization_name/nodes HTTP/1.1
	  Accept: application/json
	  Accept-Encoding: gzip;q=1.0,deflate;q=0.6,identity;q=0.3
	  X-Ops-Sign: algorithm=sha1;version=1.0;
	  X-Ops-Userid: user_id
	  X-Ops-Timestamp: 2013-03-12T17:13:28Z
	  X-Ops-Content-Hash: 2jmj7l5rfasfgSw0ygaVb/vlWAghYkK/YBwk=
	  X-Ops-Authorization-1: BE3NnHeh5yFTiT3ifuwLSPCCYasdfXaRN5oZb4c6hbW0aefI
	  X-Ops-Authorization-2: sL4j1qtEZzi/2WeF67UuytdsdfgbOc5CjgECQwqrym9gCUON
	  X-Ops-Authorization-3: yf0p7PrLRCNasdfaHhQ2LWoE/+kTcu0dkasdfvaTghfCDC57
	  X-Ops-Authorization-4: 155i+ZlthfasfhbfrtukusbIUGBKUYFjhbvcds3k0i0gqs+V
	  X-Ops-Authorization-5: /sLcR7JjQky7sdafIHNEEBQrISktNCDGfFI9o6hbFIayFBx3
	  X-Ops-Authorization-6: nodilAGMb166@haC/fttwlWQ2N1LasdqqGomRedtyhSqXA==
	  Host: api.opscode.com:443
	  X-Chef-Version: 11.4.0
	User-Agent: Chef Knife/11.4.0 (ruby-1.9.2-p320; ohai-6.16.0; x86_64-darwin11.3.0; +http://opscode.com)
	```


这里我们可以看到，被用作数字签名的属性，在请求中再次出现了，包括HTTP Method，X-Ops-Content-Hash，X-Ops-Timestamp，X-Ops-UserId。而Hash Path可以通过计算请求路径直接得到。这样Chef Server就可以用X-Ops-Sign中指定的算法，这里为SHA1，重新执行哈希过程以验证请求内容是否被第三方篡改过。

这一认证过程同样发生在用户使用Knife命令行或者Chef管理页面的时候，只是这些情况下用户无需关心认证的细节。
认证完成后，客户端就可以向Chef Server请求对各种Chef资源的访问和管理权。以Node这一资源为例，Chef Server提供了如下API，分别用于获取所有Nodes，以及对单个Node的创建、获取、修改和删除操作，如表 1 所示。

* 表 1 Chef Server提供的Node资源REST API

HTTP METHOD | 	URL	| REQUEST BODY	| RESPONSE BODY
---------|---------|-----|----
	GET |	/nodes	|	 |{"latte": "http://localhost:4000/nodes/latte" }
	POST |	/nodes |	{ "name": "latte",	"chef_type": "node", "json_class": "Chef::Node",	"attributes": {	"hardware_type": "laptop" }, "overrides": {}	"defaults": {},	"run_list": [ "recipe[unicorn]" ] }| {	"uri": "http://localhost:4000/nodes/latte" } 
	DELETE | 	/nodes/NAME |	 |	{	"overrides": {},	"name": "latte",	"chef_type": "node",	"json_class": "Chef::Node",	"attributes": { "hardware_type": "laptop" }, "run_list": [ 	"recipe[apache2]"	],	"defaults": {} } 
	GET	 | /nodes/NAME	|	| { "name": "node_name",	"chef_environment": "_default",	"run_list": [	"recipe[recipe_name]"	] 	"json_class": "Chef::Node",	"chef_type": "node",	"automatic": { ... },	"normal": { "tags": [ ] },	"default": { },	"override": { } }
	PUT |	/nodes/NAME | 	{ "overrides": {},	"name": "latte",	"chef_type": "node",	"json_class": "Chef::Node",	"attributes": {	"hardware_type": "laptop"	},	"run_list": [	'recipe[cookbook_name::recipe_name],	role[role_name]'	],	"defaults": {}	} | 	Response codes:	200 OK 	401 Unauthorized	403 Forbidden	404 Not Found	413 Request entity too large

	
通过调用Chef的REST API，就可以完成对Chef资源的管理。

#### 调用 Chef 命令

为了实现上层系统调用Chef的完全自动化，有时候需要自动化Chef Client的配置。

将一个非Chef Client的普通机器自动注册成Chef Client，能提高整个系统的自动化程度。

Chef 的REST API没有提供注册Chef Client的功能，而Chef的bootstrap命令是用来完成这个工作的。

所以上层系统需要使用某些机制（如使用 JSch）来在Chef Workstation上运行 bootstrap 命令。

通常我们用如下命令来将一个普通机器注册成Chef Node：

```
knife bootstrap client_IP -x username -P password
```

如果客户端已经安装了chef-client 软件，此命令会直接将这个客户端注册成一个Chef Node；如果客户端没有安装 chef-client软件，此命令会试图从网络上直接下载安装包去安装chef-client然后注册。

对于没有外部网络连接的客户端，又没有安装chef-client软件，我们可以自定义bootstrap所用的模板，让其不从网络下载 chef-client，而直接从本地服务器下载chef-client进行安装（前提是配置一个本地服务器如 HTTP 服务器，将chef-client软件预先放到服务器上）。

如果是默认安装，bootstrap所用的模板位于Workstation的/opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-11.10.4/lib/chef/knife/bootstrap/目录下。

我们可以在此目录下新建一个模板，命名为ubuntu12.04-gems-mine，在原有的ubuntu12.04-gems模板基础上进行修改，如将安装chef-client软件的部分自定义为清单 5 所示。

* 清单 5.自定义 bootstrap 模板

	```
	if [ ! -f /usr/bin/chef-client ]; then
	  mkdir -p /tmp/chef-installer
	  cd /tmp/chef-installer
	  #Copy chef client rpm from HTTP file server
	  wget http://http_server_ip/chef/chef-11.12.2-1.el6.x86_64.rpm
	  rpm -ivh chef-*.rpm
	  rm -rf /tmp/chef-installer
	fi
	```

然后我们使用自定义的模板来运行 bootstrap 命令：

```
# knife bootstrap client_ip -x username -P password –d ubuntu12.04-gems-mine
```

这样在 bootstrap 一个普通的没有安装 chef-client 的机器时，Chef就会从本地的服务器上下载chef-client软件（无需外部网络连接），安装在客户机上，然后注册成Chef Node。

可见，对于没有提供Chef REST API的一些特殊Chef功能，可以通过调用Chef的命令行来完成。

#### Chef 的异常处理机制

Chef Client运行完相关配置后，运行结果是成功还是失败，成功或者失败之后怎么处理，这些在集成Chef的系统中非常重要。

一方面，上层系统需要监控Chef的运行状态；另外一方面，对于Chef的运行结果，上层系统一般需要做一些处理。

为此，Chef提供了两种类型的Handler来分别处理失败和成功的运行结果：Exception Handler 和 Report Handler。Chef提供一个基础的chef_handler资源，我们可以自定义自己的Handler来支持业务需求。

自定义Handler 需要继承Chef提供的基础Handler类。下面我们自定义一个调用某业务REST API的Handler名为"CallRestAPI"：

* 清单 6.自定义 Chef Handler require 'chef/handler'
	
	```
	require 'net/http'
	 
	class CallRestAPI < Chef::Handler
	  def initialize(config = {})
	    @uri = config[:_uri]
	    @username = config[:_username]
	    @password = config[:_password]
	    @ossreqid = config[:_ossrequestid]
	  end  
	 
	  def report
	    if run_status.success?
	      status = "Success"
	      extra = ""
	    else
	      status = "Failed"
	      extra = "#{run_status.exception}"
	    end
	     
	uri = @uri
	username = @username
	password = @password
	ossreqid = @ossreqid
	     
	    begin
	    uri = URI(uri)
	    http = Net::HTTP.new(uri.host, uri.port)
	    url = "/api/auth"
	    data = "{\"username\":\"#{username}\",\"password\":\"#{password}\"}"
	    header = {'Content-Type' => 'application/json'}
	    resp = http.post(url, data, header)
	    mytoken = resp.body
	    url = "/api/requests?requestId=#{ossreqid}&status=#{status}&message=#{extra}"
	    header = {'Cookie' => " mytoken =#{mytoken}"}
	    resp = http.send_request('PUT', url, nil, header)       
	    rescue => e
	      Chef::Log.error("Error call rest api: #{e}")
	end
	 
	  end
	 
	end
	```

在这个自定义的Handler中，首先初始化传来的参数。然后根据Chef内置的变量（run_status）来判断运行结果是成功还是失败，如果失败，还可以得到失败的异常消息。

接着就是调用具体业务的REST API将此结果返回。对于这样一个自定义的Handler，我们可以将其作为一个文件放到一个 Cookbook的files的default目录下（假设命名为chef-handler-mine.rb），然后在此Cookbook的Recipe中启动这个 Handler。启动示例如下：

* 清单 7.启动（调用）自定义 Chef Handler include_recipe "chef_handler"

	```
	cookbook_file "#{node['chef_handler']['handler_path']}/chef-handler-mine.rb" do
	  source "chef-handler-mine.rb"
	  mode "0755"
	end.run_action(:create)
	 
	chef_handler "CallRestAPI " do
	  source "#{node['chef_handler']['handler_path']}/chef-handler-mine.rb"
	  arguments [
	        :_uri           => node['uri'],
	        :_username      => node['username'],
	        :_password      => node['password'],
	        :_ossrequestid      => node['ossrequest_requestid'],
	            ]  
	  supports :report => true, :exception => true
	  action :enable
	end.run_action(:enable)
	```

在清单 7中，首先将自定义的Handler传到Chef Client端，然后启动这个Handler。启动之后，当此Recipe运行完成之后，无论成功失败（因为将report和exception两者都设置成了true），都会自动调用这个自定义Handler，将结果返回给上层业务系统。

如果多个Cookbook需要相同的Handler机制，我们可以将这样的Handler抽取出来写成一个单独的Cookbook，比如，创建一个名为myhandler的Cookbook，然后将自定义的"CallRestAPI"Hander放到files的default目录下，再将清单 7中的代码写到myhandler的default的Recipe中。在其他需要调用这个Handler的Cookbook里，只需要加上include_recipe "myhandler"就可以了。

## 结束语

本文介绍了 Chef 环境的安装与使用方法，主要是根据实际经验来介绍的，文中更多的是举例和成功实践。如果要全面详细的了解Chef的各个组件如何配合工作，Chef提供了哪些内置的资源方便使用者开发Cookbook，可以参考Chef的官网。

在当前大势所趋的云环境中，自动化部署也是其中一个重要工作，对于虚机的部署可以调用相关的虚拟化平台接口，而对于虚机的配置，以及软件和应用的自动化部署，就能用到Chef这个自动化工具了，而事实也证明，Chef在当下正变得越来越流行，已被很多企业级云平台采用，相信它会越来越广泛地应用到自动化领域中。



**参考资料：** [IBM chef](https://www.ibm.com/developerworks/cn/cloud/library/1407_caomd_chef/)

![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注该公众号 