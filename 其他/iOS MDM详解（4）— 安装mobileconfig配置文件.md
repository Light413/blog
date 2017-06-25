###简介
>配置文件的安装有以下几种方式：

> * 方式一、使用 ` Apple Configurator 2`安装
> * 方式二、通过邮件的方式
> * 方式三、通过网页的方式
> * 方式四、通过`over-the-air`的方式
> 
>这里我们使用了方式三来安装。配置文件的安装经历三个过程：通过网页访问下载文件、根据提示安装，设备认证过程，设备更新Token信息的过程。


### 设备认证
主动以PUT 请求的方式访问 `CheckInURL`提交设备相关的信息，发送的内容如下：


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>BuildVersion</key>
	<string>13D15</string>
	<key>IMEI</key>
	<string>35 884805 093285 4</string>
	<key>MEID</key>
	<string>35884805093285</string>
	<key>MessageType</key>
	<string>Authenticate</string>
	<key>OSVersion</key>
	<string>9.2.1</string>
	<key>ProductName</key>
	<string>iPad4,5</string>
	<key>SerialNumber</key>
	<string>F4KMG0FSFLMM</string>
	<key>Topic</key>
	<string>com.apple.mgmt.External.*</string>
	<key>UDID</key>
	<string>UDID</string>
</dict>
</plist>
```
以上可看出

`MessageType`标记消息类型，其值为`Authenticate`<br>
`Topic`推送主题，即证书中的用户ID<br>
`UDID`设备的唯一标示符

Server收到请求后根据`MessageType`的值做不同的数据处理操作，然后响应一个空的字典,完成认证

```
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN""http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
		<dict></dict>
	</plist>

```

###设备发送TokenUpdate信息

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>AwaitingConfiguration</key>
	<false/>
	<key>MessageType</key>
	<string>TokenUpdate</string>
	<key>PushMagic</key>
	<string>2969ACF9-DD9C-46D2-8784-F0949CB25BB9</string>
	<key>Token</key>
	<data>
	m200tX8dSj/oBDKKlBpy1NRTQzvfOLNYa1rB7A0/rUM=
	</data>
	<key>Topic</key>
	<string>com.apple.mgmt.External.bc2c8764-9ce5-4fd3-9330-4036325a91cc</string>
	<key>UDID</key>
	<string>233deb277d03bd4aaf91108390c7d9fe2c49c8be</string>
	<key>UnlockToken</key>
	<data>
	REFUQQAABO...//Base64编码的字符串，锁屏时需要的参数
	</data>
</dict>
</plist>

```
主要参数：
>`PushMagic` ：MDM server 用于推送时标记设备唯一的识别符（可以理解为类似token），每次与APNs发消息时必须带上它。

>`Token` ：设备的token。

>`UnlockToken`当清除设备密码时需要的一个token，必须带上。

Server响应，返回的数据为空，操作完成结束连接。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Cache-Control: no-cache
Content-Type: text/plain;charset=UTF-8
Content-Length: 0
Date: Wed, 26 Apr 2017 07:33:48 GMT
```

以上及完成了设备了登记注册，此时在Server后台可以查看到该注册设备相关的信息。


