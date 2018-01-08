# Yunba RESTful API 使用手册


## 注册开发者账号

打开 [云巴官方网站](https://yunba.io)，点击注册创建账号。

![create_accout.png](https://raw.githubusercontent.com/yunba/docs/master/image/productpng_portal_register_account.png)

## 在云巴 Portal 上创建新应用

注册账号成功后，页面跳转到 “我的应用列表” 界面。点击 “创建应用”，输入 “应用名称” 和 “应用包名”。


创建成功后，会获得与 “应用包名”一一对应的 [AppKey](product_kb_app_key.md)，可在 “应用详情” 页面查看。
在下文的示例中，将 appkey 和 seckey 替换为您从 [Portal](product_kb_portal.md) 获取到的 AppKey 和 Secret Key，即可运行。


>**注**：请妥善保管好您的 AppKey 和 Secret Key，不要泄露给他人。

![productpng_portal_creat_application.png](https://raw.githubusercontent.com/yunba/docs/master/image/productpng_portal_creat_app.png)

## 方法

>***注：msg 字段使用了 [URL encoding](https://www.w3schools.com/tags/ref_urlencode.asp)，特殊符号请遵循该标准，如百分号 '%' 的 URL encoding 为 '%25'***

### HTTP GET

**重要提示：GET 方法仅供测试，实际应用中请使用 POST 方法。（[GET 与 POST 的比较](http://www.w3school.com.cn/tags/html_ref_httpmethods.asp)）**

**格式**

```url
http://rest.yunba.io:8080?method=<method>&appkey=<app-key>&seckey=<secret-key>&topic=<topic>&msg="your message"
```
可直接在浏览器地址栏中打开 URL，或使用 cURL 命令来执行。其中，method 的详细说明见下文的 HTTP POST 章节。


在使用 `publish_to_alias` 方法时，请用 alias=<alias> 替换 topic=<topic> 即可。

**示例**

```bash
$curl  --request GET "http://rest.yunba.io:8080?method=publish&appkey=567a4a754407a3cd028aaf6b&seckey=sec-mj64xlu0ob1Xs1wWuZzmGZOYZqrpFmFxp5jHULr13eUZCVpS&topic=news&msg="good_news""
```
**注**：上文给出的 AppKey 和 SecretKey 功能受限，仅供文档举例使用。

### HTTP POST 

**格式**

对于 HTTP POST 请求，请求的 body 使用 JSON 格式，请将 HTTP header 的 Content-Type 设为 application/json。


请求的 JSON 格式及参数说明如下。


同样地，在使用 `publish_to_alias` 时，请用 "alias":<alias> 替换 "topic":<topic> 即可。

```json
{
	"method":<method>, 
	"appkey":<app-key>, 
	"seckey":<secret-key>, 
	"topic":<topic>, 
	"msg":<message>,

	"opts":	{
				"time_to_live":<number>,
				"qos":<number>,
				"apn_json":	{
								"aps":	{
											"alert":<string>,
											"badge":<number>,
											"sound":<string>,
											"content-available":<number>
								}
				},
				"third_party_push": {
								"notification_title":"你好", 
								"notification_content":"云巴推送"
								}
	}
}
```

**参数说明：**

* `time_to_live`：用来设置离线消息保留多久。单位为秒（例如，“3600”代表1小时），默认值为 3 天，最大不超过 15 天。
* `qos`：用来设置服务质量等级。有三种取值：“0”表示最多送达一次；“1”表示最少送达一次；“2”表示保证送达且仅送达一次。默认为 1 。详见云巴知识库的 [QoS](product_kb_qos.md) 篇。
* `opts` 为可选项。
	- 如果不带有 opts 参数，会发送默认的 APN。
	- 如果带有 opts 参数，且 opts 中出现了 apn_json (aps) 项，就会发送 APN。
	- **如果带有 opts 参数，且 opts 中没有出现 apn_json (aps) 项，就不会发送 APN。**
	- **如果不带 opts 或 opts 中没有 third_party_push 项，就不会发送小米／华为推送。**


下面是一个带有 `apn_json` 的 `opts` 参数示例，详细的参数说明请参考 [APNs 的官方文档](https://developer.apple.com/library/prerelease/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/PayloadKeyReference.html#//apple_ref/doc/uid/TP40008194-CH17-SW1)。

```json
"opts":	{
	"qos": 1,
	"time_to_live":36000,
	"apn_json":{
		"aps":	{
				"alert":"yunba",
				"badge":3,
				"sound":"bingbong.aiff"
			}
	}
}
```

## method 说明及示例

目前支持的 method 包括：
* `publish`
* `publish_to_alias`
* `publish_to_alias_batch`

其中，`publish`、`publish_to_alias`和`publish_to_alias_batch`可以带 opts 参数，带上参数后，就相当于 `publish2`、`publish2_to_alias`和`publish2_to_alias_batch`。


下面逐一介绍这几种 method，并给出示例。


将下文示例中的 appkey 和 seckey 替换为您从 Portal 获取到的 AppKey 和 Secret Key，即可直接运行。
运行后，会收到返回的状态信息，详见文末的状态参数说明。

### `publish`

用于向指定的频道（topic）发送消息，所有订阅了该频道的客户端都可以收到消息。

```bash
$ curl -l -H "Content-type: application/json" -X POST -d '{"method":"publish", "appkey":"567a4a754407a3cd028aaf6b", "seckey":"sec-mj64xlu0ob1Xs1wWuZzmGZOYZqrpFmFxp5jHULr13eUZCVpS", "topic":"news", "msg":"good news"}' http://rest.yunba.io:8080
```

### `publish_to_alias`

用于向指定的别名（alias）发送一对一的消息。

```bash
$ curl -l -H "Content-type: application/json" -X POST -d '{"method":"publish_to_alias", "appkey": "567a4a754407a3cd028aaf6b", "seckey":"sec-mj64xlu0ob1Xs1wWuZzmGZOYZqrpFmFxp5jHULr13eUZCVpS", "alias":"test", "msg":"message from RESTful API", "opts":{"time_to_live":20000}}' http://rest.yunba.io:8080
```

### `publish_to_alias_batch`

用于同时向多个指定的别名（alias）发消息。群发的别名数目最好控制在 1000 以下。


例如，我们向别名为 Jack 和 Rose 的客户端同时发送消息。

```bash
$ curl -l -H "Content-type: application/json" -X POST -d '{"method":"publish_to_alias_batch", "appkey":"567a4a754407a3cd028aaf6b", "seckey":"sec-mj64xlu0ob1Xs1wWuZzmGZOYZqrpFmFxp5jHULr13eUZCVpS", "aliases":["Jack","Rose"], "msg":"good news", "opts":{"time_to_live": 20}}' http://rest.yunba.io:8080
```

发送后，返回如下。参考文末的返回状态说明可以看出，给 Jack 的消息发送成功了。但由于该 AppKey 下并不存在别名为 Rose 的客户端，向 Rose 发的消息发送失败了。

```bash
{"status":0,"results":{"Jack":{"status":0,"messageId":512625795122860032},"Rose":{"status":5,"alias":"567a4a754407a3cd028aaf6b-Rose","error":"alias not found"}}}
```


## 支持 https

云巴对付费用户支持 https。目前，免费用户可以试用 RESTful 和 socket.IO 的加密链路，实际生产使用需要付费。

**注意，使用 SSL/TLS 方式进行连接，port 为 443；否则，port 为 3000。**



## 发送状态回复

**发送成功**

```json
{"status":0, "messageId": "<message-id>"}
```

**参数错误**

```json
{"status":1, "error": "invalid parameters"}
```

**内部服务错误**

```json
{"status":2, "error": "internal server"}
```

**没有应用**

```json
{"status":3}
```

**发布超时**

```json
{"status":4, "error": "timeout"}
```

**没有找到 Alias**
 
```json
{"status":5, "alias":"567a4a754407a3cd028aaf6b-test", "error": "alias not found"}
```
**[常见问题](restful_faq.md)**
