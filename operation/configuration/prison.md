# 配置限流
本章介绍如何配置限流功能。该功能将限制特定请求的速率，以对后端服务进行保护。

## 开启限流模块
在conf/bfe.conf中，打开该模块
```ini
Modules = mod_prison
```

## 模块配置
模块配置在目录conf/mod_prison/中，包含两个文件：

```
# ls
mod_prison.conf	prison.data
```

与其他模块配置相似，mod_prison.conf为模块基础配置文件，指向prison规则文件。
```ini
[basic]
ProductRulePath = mod_prison/prison.data
```

示例的prison.data如下:

```json
{
	"Version": "1",
	"Config": {
		"example_product": [{
			"Name": "example_prison",
			"Cond": "req_path_prefix_in(\"/prison\", false)",
			"accessSignConf": {
				"url": false,
				"path": false,
				"query": [],
				"header": [],
				"Cookie": [
					"UID"
				]
			},
			"action": {
				"cmd": "CLOSE",
				"params": []
			},
			"checkPeriod": 10,
			"stayPeriod": 10,
			"threshold": 5,
			"accessDictSize": 1000,
			"prisonDictSize": 1000
		}]
	}
}
```
上述示例对路径为“/prison”的请求进行限流。其中，"accessSignConf"指示了限制的流量的维度，具体见后续描述。本例子中，将统计cookies中的"UID"，限制不同"UID"的访问流量。

## 限制特定维度的流量

通过accessSignConf字段，我们可以指定请求统计的维度，判断统计值是否达到限流阈值。
可配置的维度包括：

* UseClientIP

对请求按clientIp进行统计做限流。可以限制每个clientIP的请求速度。

* UseUrl：

对请求按URL进行统计。可以限制对每一个URL的请求速度。

* UseHost

对请求按Host进行统计。可以限制对每一个Host的请求速度。

* UsePath

对请求按Path进行统计。可以限制对每一种Path的请求速度。

* UrlRegexp

对请求的URL做正则匹配，以匹配结果为维度进行统计。可以实现按URL中的部分内容来进行限流。

* header

以请求的特定Header为维度来做统计流量。可以限制该Header所标识的每一种消息的请求速度。

* Cookie

以请求中的特定Cookie为维度进行统计。可以限制该Cookie标识的每一种请求的请求速度。比如，如果我们用Cookie中的UID来标识不同用户，可以通过指定UID，来限制每个用户的访问速度。

* query

以指定的query为维度，统计请求量来做限流。

* UseHeaders

以每个请求中的所有Header为维度来进行统计(合并一个消息的所有header)。限制不同Header的每一种消息的请求速度。

## 设置限流门限

通过下述参数配置限流：

* checkPeriod

设置统计周期，单位秒。

* stayPeriod

被限流后的惩罚时长。在该时间段内，该维度访问请求都将被限制。

* threshold

限制的阈值。一个维度的统计数量达到该阈值将触发限流。

* accessDictSize

访问统计表的大小。统计表保存了当前配置的维度的所有统计值。比如，如果以ClientIp为维度进行的统计，该表包含每个ClientIp的访问量。

* prisonDictSize

访问封禁表的大小。按维度统计后，每类命中限流的请求，都会被保存在封禁表中。所以，封禁表保存了当前所有处于封禁状态的某类请求。比如某频繁访问，被限流的IP地址等。

## 设置限流动作

某类请求命中限流规则后，可以在"action"中配置对该类请求的后续动作。
* CLOSE

直接关闭请求的连接。

* PASS

正常转发，不做任何处理。

* FINISH

响应后关闭连接。

* REQ_HEADER_SET

正常转发，在请求header中插入指定key和value。key和value在params中指定。
