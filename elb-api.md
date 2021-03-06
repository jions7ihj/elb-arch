# ELB api文档

## 目录
* 说明
* 用户API -- 操作类
	* CreateLoadBalancer
	* DeleteLoadBalancer
	* ConfigureHealthCheck
	* CreateLBCookieStickinessPolicy
	* CreateLoadBlanacerPolicy
	* DeleteLoadBalancerPolicy
	* RegisterInstancesWithLoadBalancer
	* DeregisterInstancesFromLoadBalancer
	* ModifyLoadBalancerAttributes
	* CreateLoadBalancerListeners
	* DeleteLoadBalancerListners
	* CheckLoadBalancerStatus
	* AttachLoadBalancerToSubnets
	* DettachLoadBalancerFromSubnets	
	
* 用户API -- 描述类
	* DescribeLoadBlanacerAttributes
	* DescribeInstanceHealth
	* DescribeLoadBalancerPolicies
	* DescribeLoadBalancerPolicyTypes
	* DescribeLoadBalancers
	 
* 管理员API -- 操作类
	* RegisterTengineWithLoadBalancer
	* DeregisterTengineFromLoadBalancer
	* UpdateAgentForLoadBalancer
	* UpdateScriptForLoadBalancer
	* ExecCommandForLoadBalancer
		
* 管理员API -- 描述类	
	* DescribeLoadBalancerServers
	* DescribeLoadBalancerAgents
	* DescribeLoadBalancerLVSConf
	* DescribeLoadBalancerTengineConf
	
## 一、说明

* 目前所有API不做auth校验，只在api server上设白名单
* 所有请求走https， 默认域名 https://elb.cloud.netease.com/api
* 所有返回结果都是json
* 所有响应结果里都带了requestId，参考了aws，主要为了日志跟踪，所有请求都有唯一的id
* 有很多http请求响应需要的时间非常久，如果等待返回结果肯定会引起http超时。因此一般发请求后立刻返回，通过 CheckLoadBalancerStatus API检查LoadBlancer的状态


## 二、用户API -- 操作类

### 2.1 CreateLoadBalancer

#### 描述

创建一个负载均衡器。

一个完整的负载均衡器需要较长的周期来创建，且需要非常多的参数。

LoadBalancer的参数有：
ELB名称、Listener配置（LB的协议和端口，后端实例的协议和端口）、Health Check配置(Ping的协议和端口，Ping的间隔和超时时间，HTTP检查域名、检查端口、检查路径、响应超时时间，鉴定健壮和非健壮的标准)、会话保持、均衡策略、ELB管理的主机实例配置等。

参考amazon的创建步骤，本请求主要是创建listener和负载均衡相关数据。

HealthCheck, 会话保持， 实例管理等将分为多个API请求完成。

#### 请求参数

* LoadBalancerName： 名称
* Listeners.member.N： listener的列表， 是一个三元组 {LoadBalancerPort,InstancePort,Protocol}

#### 响应结果

* LoadBalancerUrl:  
	* 负载均衡器对应的URL
	* Type: String

#### 错误

##### DuplicateAccessPointName
	
* 名称重复
* HTTP Status Code: 400

##### InvalidateConfigurationRequest

* 请求的配置不正确
* HTTP Status Code: 409

##### InvalidateScheme

* 传入Scheme错误
* HTTP Status Code: 409

##### CertificateNotFound

* 使用SSL证书时证书未找到
* HTTP Status Code: 409


####  示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CreateLoadBalancer
&Listeners.member.1.LoadBalancerPort=443&Listeners.member.1.InstancePort=443&Listeners.member.1.Protocol=https &Listeners.member.1.InstanceProtocol=https 
&Listeners.member.1.SSLCertificateId=arn:aws:iam::123456789012:server-certific ate/servercert
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"LoadBalancerUrl" : "https://10.10.12.12"	
}
```

* 示例错误响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"error" : "DuplicateAccessPointName"	
}

```



### 2.2 DeleteLoadBalancer

#### 描述

删除负载均衡器。

#### 请求参数

* LoadBalancerName: 负载均衡器名称

#### 响应结果

无，只要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeleteLoadBalancer
&LoadBalancerName=blog

```
* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}
```


### 2.3 ConfigureHealthCheck

#### 描述

配置健康检查的参数

#### 请求参数

* LoadBalancerName: 负载均衡名
* HealthCheck: 健康检查的参数信息，包括以下字段：


#### 响应结果

* HealthCheck： 更新新后的健康检查结果

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=ConfigureHealthCheck
&LoadBalancerName=blog
&HealthCheck.HealthyThreshold=2
&HealthCheck.HealthyThreshold=2
&HealthCheck.Target=HTTP:80/ping
&HealthCheck.Interval=30&HealthCheck.Timeout=3

```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"HealCheck": {
		"Integerval": 30,
		"Target": HTTP:80/ping,
		"HealthyThreshold":2,
		"Timeout":3,
		"UnhealthyThreshold":2
	}
}

```



### 2.4 CreateLBCookieStickinessPolicy

#### 描述

创建session sticky的policy。

#### 请求参数

* LoadBalancerName: 负载均衡名
* CookieExpirationPeriod: cookie超时时间，默认会持续到浏览器关闭
* CookieName：负载均衡基于cookie名， 默认为jsessionid
* PolicName：策略名


#### 响应结果

只需要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### DuplicatePolicyName

* 策略名重复
* HTTP Status Code: 400

##### InvalidConfigurationRequest

* 请求的配置参数不正确
* HTTP Status Code: 409


#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CreateLBCookieStickinessPolicy
&LoadBalancerName=blog
&CookieName=xsessionid
&PolicyName=blogcookiePolicy

```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}

```

### 2.5 CreateLoadBalancerPolicy

#### 描述

创建策略。 策略可认为是某类选项，如请求超时的选择、tengine选项， LVS选项等。如：

* SSLNegotiationPolicyType
* BackendServerAuthenticationPolicyType
* PublicKeyPolicyType
* LBCookieStickinessPolicyType
* ProxyProtocolPolicyType

#### 请求参数

* LoadBalancerName: 负载均衡名
* PolicName：策略名
* PolicyTypeName： 对应的策略类型
* PolicyAttributes.member.N: 一系列的策略参数


#### 响应结果

只需要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### DuplicatePolicyName

* 策略名重复
* HTTP Status Code: 400

##### InvalidConfigurationRequest

* 请求的配置参数不正确
* HTTP Status Code: 409

##### PolicyTypeNotFound* PolicyType不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CreateLoadBalancerPolicy
&Name=ProxyProtocol&PolicyAttributes.member.1.AttributeValue=true &PolicyTypeName=ProxyProtocolPolicyType 
&LoadBalancerName=my-test-loadbalancer&PolicyName=EnableProxyProtocol
```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}

```

### 2.6 DeleteLoadBalancerPolicy

#### 描述

删除策略。 

#### 请求参数

* LoadBalancerName: 负载均衡名
* PolicyName: 策略名称

#### 响应结果

只需要requestId

#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### DuplicatePolicyName

* 策略名重复
* HTTP Status Code: 400

##### PolicyNameNotFound* PolicyName不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeleteLoadBalancerPolicy
&LoadBalancerName=my-test-loadbalancer&PolicyName=EnableProxyProtocol
```
* 示例响应

```

{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}

```

### 2.7 RegisterInstanceWithLoadBalancer


#### 描述

往负载均衡器添加后端服务实例。

#### 请求参数

* Instances.member.N： instanceID列表。在网易应该是云主机的id
* LoadBalancerName：负载均衡器名称

#### 响应结果

* Instances： 更新的instance列表


#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=RegisterInstanceWithLoadBalancer
&LoadBalancerName=my-test-loadbalancer&Instances.memeber.1.InstanceId=i-315b7e51
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"Instances": [
		{"InstanceId": "i-315b7e51"},
		{"InstanceId": "i-315b7e87"},
	]
}
```

### 2.8 DeregisterInstanceFromLoadBalancer


#### 描述

从负载均衡器添移除后端服务实例。

#### 请求参数

* Instances.member.N： instanceID列表。在网易应该是云主机的id
* LoadBalancerName：负载均衡器名称

#### 响应结果

* Instances： 更新的instance列表


#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeregisterInstanceFromLoadBalancer
&LoadBalancerName=my-test-loadbalancer&Instances.memeber.1.InstanceId=i-315b7e51
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"Instances": [
		{"InstanceId": "i-315b7e87"}
	]
}
```

### 2.9 ModifyLoadBalancerAttributes

#### 描述 

修改负载均衡器的相关属性

#### 请求参数

* LoadBalancerName：负载均衡器名称
* LoadBalancerAttributes： 负载均衡器的属性，包括以下几项：
	* AccessLog：打开请求日志
	* ConnectionDrain: 等请求都发完后才将请求从要移除的实例移除
	
#### 响应结果

 * LoadBalancerName: 负载均衡器名称
 * LoadBalancerAttributes：负载均衡器的属性
 
#### 错误
	
##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### InvalidConfigurationRequest

* 请求配置的更改有错
* * HTTP Status Code: 409

##### LoadBalancerAttributeNotFound

* 请求的属性不存在
* HTTP Status Code: 400

## 示例


* 示例请求

```
https://elb.cloud.netease.com/api?action=ModifyLoadBalancerAttributes
&LoadBalancerName=my-test-loadbalancer&LoadBalancerAttributes.AccessLog.Enabled=true&LoadBalancerAttributes.AccessLog.S3BucketName=my-loadbalancer-logs&LoadBalancerAttributes.AccessLog.S3BucketPrefix=my-bucket-prefix/prod&LoadBalancerAttributes.AccessLog.EmitInterval=60
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"AccessLog": {
		 "Enabled": true,        "S3BucketName": "my-loadbalancer-logs",        "S3BucketPrefix": "my-bucket-prefix/prod",        "EmitInterval": 60
	}
}

```

### 2.10 CreateLoadBalancerListners

#### 描述 

在负载均衡器上创建一个或多个listner。 支持https

#### 请求参数

* LoadBalancerName 负载均衡器名字
* Listners.member.N 端口号列表

#### 响应参数

* 没有, 只要返回requestId

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### DuplicateListner

* Listner对应端口已经存在
* HTTP Status Code: 400

#### InvalidConfigurationRequest

* 请求的配置修改错误
* HTTP Status Code: 409

#### CertificateNotFound

* 证书未找到
* HTTP Status Code: 400

#### 示例

* 示例请求 

```
https://elb.cloud.netease.com/api?action=CreateLoadBalancerListners
&Listeners.member.1.LoadBalancerPort=443 
&Listeners.member.1.InstancePort=443 
&Listeners.member.1.InstanceProtocol=https 
&Listeners.member.1.SSLCertificateId=arn:aws:iam::123456789012:server-certificte/servercert&LoadBalancerName=MyHTTPSLoadBalancer
```
* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"
}
```


### 2.11 DeleteLoadBalancerListeners

#### 描述

 删除某指定端口的负载均衡器listener
 
#### 请求参数

* LoadBalancerName 负载均衡器名称
* LoadBalancerPorts.member.N  端口号列表

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

### 2.12 CheckLoadBalancerStatus

#### 描述

检查负载均衡器的状态。很多操作执行周期长，有很多中间状态， 需要通过这个API去查

#### 请求参数

* LoadBalancerName 负载均衡器名称

#### 响应参数

* LoadBalancerStatus 负载均衡器的创建状态
InCreateProcess InDeleteProcess InConfigListenerProcess Complete 

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=CheckLoadBalancerStatus
&LoadBalancerName=MyLoadBalancer

```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
    "LoadBalancerStatus" : {
    	"status": "InConfigListenerProcess"    
    }
}

```

### 2.13 AttachLoadBalanerToSubnets

下期完成

### 2.14 DettachLoadBalanerFromSubnets

下期完成


## 三、用户API -- 描述类

### 3.1 DescribeLoadBlanacerAttributes

#### 描述

显示负载均衡器的相关属性

#### 请求参数

* LoadBalancerName：负载均衡器名称

#### 响应结果

* LoadBalancerAttributes： 负载均衡器相关属性

#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例


* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBlanacerAttributes
&LoadBalancerName=my-test-loadbalancer```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",
	"LoadBalancerAttributes" : 
	{
		"AccessLog": {
		  "Enabled": true,      	  "S3BucketName": "my-loadbalancer-logs",      	  "S3BucketPrefix": "my-bucket-prefix/prod",      	  "EmitInterval": 60
		},
		"ConnectionDraining": {
			"Enabled": true,
			"timeout": 60
		}
	}}
```


### 3.2 DescribeInstanceHealth

#### 描述

显示负载均衡器后端instance的健康状态。

#### 请求参数

* LoadBalancerName；负载均衡名
* PolicyNames.member.N： 策略参数列表

#### 响应参数

* instanceStates： 各个instance列表

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### InvalidEndPoint

* endpoint 不正确
* HTTP Status Code: 400


#### 示例


* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeInstanceHealth
&LoadBalancerName=my-test-loadbalancer
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",	"InstanceStates" :
	[
		{
			"Description": "N/A",
			"InstanceId": "i-90d8c2a5",
			"State": "InService",
			"ReasonCode": "N/A"
		},
		{
			"Description": "Instance has failed at least the UnhealthyThreshold number of health checks consecutively.",
			"InstanceId": "i-90d8c2c9",
			"State": "OutOfService",
			"ReasonCode": "Instance"
		}
	]}
```



### 3.3 DescribeLoadBalancerPolicies

#### 描述

描述负载均衡器的策略列表。

#### 请求参数

* LoadBalancerName；负载均衡名
	* Type: String
	* Required: No 
* PolicyNames.member.N： 策略参数列表名称
	* Type: String
	* Required: No 

#### 响应参数

* PolicyDescriptions： 策略描述列表

#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### PolicyNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBalancerPolicies
&LoadBalancerName=my-test-loadbalancer
```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",	"PolicyDescriptions" :
	[
		{
			"PolicyName": "MyDurationStickyPolicy",
			"PolicyTypeName": "LBCookieStickinessPolicyType",
			"PolicyAttributeDescriptions": [
				{"AttributeName" : "CookieExpirationPeriod","AttributeValue": 60}
			]
		},
		{
			"PolicyName": "EnableProxyProtocol",
			"PolicyTypeName": "ProxyProtocolPolicyType",
			"PolicyAttributeDescriptions": [
				{"AttributeName" : "ProxyProtocol","AttributeValue": true}
			]
		}
	]}

```


### 3.4 DescribeLoadBalancerPolicyTypes

#### 描述

描述负载均衡器的策略列表。

#### 请求参数


* PolicyNames.member.N： 策略参数列表名称
	* Type: String
	* Required: No 

#### 响应参数

* PolicyTypeDescriptions： 策略类型描述列表

#### 错误 

##### PolicyTypeNotFound

* 请求的PolicyType不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBalancerPolicyTypes```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",	"PolicyTypeDescriptions" :
	[
		{"PolicyTypeName": "SSLNegotiationPolicyType" ...}, 
		{"PolicyTypeName": "PublicKeyPolicyType" ...}, 		{"PolicyTypeName": "LBCookieStickinessPolicyType" ...}, 
		{"PolicyTypeName": "BackendServerAuthenticationPolicyType" ...}, 
		{
			"PolicyTypeName": "ProxyProtocolPolicyType",
			"PolicyAttributeTypeDescriptions": [
				{"AttributeName" : "ProxyProtocol", "AttributeType":"Boolean", "Cardinality": "ONE"}
			]
						"Description": "Policy that controls whether to include the IP address and 				port of the originating request for TCP messages.        		This policy operates on TCP/SSL listeners only"		}
	]}

```


### 3.5 DescribeLoadBalancers


#### 描述

获取所有的负载均衡器描述

#### 请求参数

* LoadBalancerNames.member.N: 负载均衡器列表， not required


#### 响应参数

* LoadBalancerDescriptions: 负载均衡器描述列表


#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBalancers

```

* 示例响应

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",	"LoadBalancerDescriptions" :
	[
		{
			"LoadBalancerName": "SSLNegotiationPolicyType",
			"CreatedTime": "2013-05-24T21:15:31.280Z",
			"HealthCheck": {
				"Interval": 90,
				"Target":"HTTP:80",
				"HealthyThreshold":2,
				"Timeout":60,
				"UnhealthyThreshold":10
			},
			"ListenerDescrpitions": [
				"listener": {
					"Protocol": "http",
					"LoadBalancerPort": 80,
					"InstanceProtocol":"HTTP",
					"InstancePort":80
				}
			],
			"instances": [
				{"InstanceId": "i-e4cbe38d"}
			],
			"policies": [
				{"LBCookieStickinessPolicies":""},
				{"OtherPolicies":""}
			]			
		}
	]}
```

## 四、管理员API -- 操作类

### 4.1 RegisterTengineWithLoadBalancer

#### 描述

在http的负载均衡器上新增tengine实例.

#### 请求参数

* LoadBalancerName：负载均衡器名称 
* TengineNumber: 添加tengine的台数


#### 响应

* tengineInstances: 更新后的instance列表

#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=RegisterTengineWithLoadBalancer&TengineNumber=2

```

* 示例响应结果

```
{
	"RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE",	"tengineInstances" :
	[
		{"InstanceId": "900088"},
		{"InstanceId": "79999"}
	]}
```


### 4.2 DeregisterTengineFromLoadBalancer

#### 描述

在http的负载均衡器上减少tengine实例

#### 请求参数

* LoadBalancerName: 负载均衡器名称
* Tengines.member.N: 删除的member列表 

#### 响应

* tengineInstances: 更新后的Instance列表


#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### TengineNotFound

* tenginge对应的instance不存在
* HTTP Status Code: 400


#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DeregisterTengineFromLoadBalancer&Tengines.member.1.InstanceId=9908899&Tengines.member.2.InstanceId=7889

```

* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "tengineInstances" :
	[
		{"InstanceId": "230000-122"},
		{"InstanceId": "u8999-1"}
	]
}

```


### 4.3 UpdateAgentForLoadBalancer

#### 描述

更新负载均衡器上所有agent的版本。

agent收到请求后，从http服务器找到最新版的agent下载并解压。  
由crontab脚本检查内存中agent版本号，发现版本低则重启。

#### 请求参数

* LoadBalancerName: 负载均衡器名称， 可不填， 不填则更新所有agent
* Instances.member.N: 某个instanceId的agent， 可不填，不填是全部
* agentVersion：agent的版本号 

#### 响应

* Instances: 更新成功的instance列表 


#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### InvalidHttpServer

* 更新的http服务器访问不了
* HTTP Status Code: 500


##### AgentVersionNotFound

* 在http服务器上，找不到对应的版本号的agent
* HTTP Status Code: 400


#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=UpdateAgentForLoadBalancer
&LoadBalancerName=uiii&agentVersion=0.3

```

* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "Instances" :
	[
		{"InstanceId": "230000-122"},
		{"InstanceId": "u8999-1"}
	]
}

```


### 4.4 UpdateScriptForLoadBalancer

#### 描述 

更新运维脚本。
运维脚本打包成一个tar包放在http服务器上， 并带有版本号， 每次更新时从服务器下载。

#### 请求参数

* LoadBalancerName  负载均衡器名， 可不填，不填则更新所有相关服务器
* Instances.memeber.N Instance名列表，可不填，不填就是全部
* scriptVersion  脚本的版本号

#### 响应

* Instances, 更新成功的服务器instance列表 

#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### InvalidHttpServer

* 更新的http服务器访问不了
* HTTP Status Code: 500

##### ScriptVersionNotFound

* 在http服务器上，找不到对应的版本号的script
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=UpdateScriptForLoadBalancer
&Instanceces.member.1=333201-1&scriptVersion=0.3

```

* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "Instances" :
	[
		{"InstanceId": "333201-1"}
	]
}

```

### 4.5 ExecCommandForLoadBalancer

#### 描述

在负载均衡器的各个主机instance上执行shell脚本。
shell脚本可以从http服务器下载， 也可以在body里post过去。

ExecCommand把宿主机上所有的运维命令都抽象了， 如更新tengine配置或权重，更新LVS配置等，都是扔shell命令完成的。

#### 请求参数

* LoadBalancerName 对应的负载均衡器
* Instances.memeber.N Instance名列表，可不填，不填就是全部
* Commmand    放在body里post过去， 与shellName 两个参数2选1 
* ShellName   执行的sh脚本名称， agent会从http服务器下载该脚本

#### 响应

* results 执行成功的instance列表，及响应结果

#### 错误 

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

##### InvalidHttpServer

* 更新的http服务器访问不了
* HTTP Status Code: 500


#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=ExecCommandForLoadBalancer
&Instanceces.member.1=333201-1&shellName=addTask.sh

```

* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "results" :
	[
		{"InstanceId": "333201-1", "response":"add cron task success"},
		{"InstanceId": "333201-2", "response":"add cron task success"},
	]
}
```

## 五、管理员API -- 描述类

### 5.1 DescribeLoadBalancerServers

#### 描述

获取负载均衡器对应所有的server信息。

#### 请求参数

* LoadBalancerName 对应负载均衡器的名称

#### 响应

* servers 所有server instances的描述

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=ExecCommandForLoadBalancer
&LoadBalancerName=xhklhi
```
* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "servers" :
	[
		{
			"InstanceId": "333201-1", 
			"ServiceType":"LVS",	
			"ContainerType":"physical",		
			"IP":"10.1.3.4",
			"port": "9200",
			"cpu": "16core",
			"memory":"32G"	
		},
		{
			"InstanceId": "333201-2", 
			"ServiceType":"Tengine",
			"ContainerType": "lxc",		
			"Weight": "0.3"	
			"IP":"10.1.3.5",
			"port": "9201",
			"cpu": "3core",			
			"memory":"32G"
		}
		{"InstanceId": "333201-2" ...}
	]
}
```

### 5.2 DescribeLoadBalancerAgents

#### 描述

获取负载均衡器对应的agent信息， 如版本号、启动时间， 占用内存、cpu，

#### 请求参数

* LoadBalancerName 对应负载均衡器的名称

#### 响应

* agents, agents列表的信息

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBalancerAgents
&LoadBalancerName=xhklhi
```
* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "agents" :
	[
		{
			"InstanceId": "333201-1", 
			"ServiceType":"LVS",	
			"AgentVersion":"0.3.1",		
			"AgentStartTime":"2014-03-12 05:13:02",
			"AgentMemory": "5M",
			"AgentCpu": "0.03"
		},
		{"InstanceId": "333201-2" ...}
	]
}
```


### 5.3 DescribeLoadBalancerLVSConf

#### 描述

获取LVS的配置信息

#### 请求参数

* LoadBalancerName 对应负载均衡器的名称

#### 响应

* LVSConf 的信息

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBalancerLVSConf
&LoadBalancerName=xhklhi
```
* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "LVSConfigs" : [
    	{
    	"InstanceId":"xxx-111",
    	"BackUpInstanceId": "yyy-222",
    	"ConnectMode":"DR"
    	"HashRoute": "rountrobin",
    	"port":9092,
    	"IP":"10.13.13.13"
    	"Realservers" : [
    		{"InstanceId": "zzz-333"}
    	]
    	}
    ]
}
```

### 5.4 DescribeLoadBalancerTengineConf

#### 描述

获取tengine的配置信息

#### 请求参数

* LoadBalancerName 对应负载均衡器的名称

#### 响应

* TengineConfigs ， tengine的列表

#### 错误

##### AccessPointNotFound

* 请求的LB不存在
* HTTP Status Code: 400

#### 示例

* 示例请求

```
https://elb.cloud.netease.com/api?action=DescribeLoadBalanceTengineConf
&LoadBalancerName=xhklhi
```
* 示例响应

```
{
    "RequestId": "06b5decc-102a-11e3-9ad6-bf3e4EXAMPLE"，
    "TengineConfigs" : [
    	{
    	   "InstanceId": "xxx-yyy",
    	   "worker_procsses":8,
    	   "ip": "10.13.12.12",
    	   "port": 80,
    	   "upstream-backend": {
    	   		"ip_hash": false,
    	   		"servers":[       	   	   
    	   		{"url":"127.0.0.1:8080 ", "weight":5},
    	   		{"url":"127.0.0.2:8080", "weight":3, "max_fails":3, "fail_timeout":"30s}"
    	   		]
    	   }
    	}
    ]
}
```
































	



	 	 