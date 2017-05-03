###简介
>给设备发个指令操作的第一步不是由MDM Server直接向APNs推送指令的，但是由Server向APNs发送一个特定的指令来好比唤醒设备，设备被唤醒之后会根据已安装的配置文件的ServerURL 的地址主动发起请求，报告自己的当前状态，只有其状态值为`Idle`设备才会接收Server指令操作。

如图所示：

![MDM工作流程](https://github.com/Light413/blog/blob/master/其他/img/MDM工作流程.jpg?raw=true)

所以完成一次指令推送经历以下过程：

* 1、server 与APNs建立连接，发送数据。
* 2、当设备收到APNs推送消息时，主动连接server报告本身的状态空闲
* 3、server收到设备发来的状态信息，发出操作命令
* 4、设备收到命令执行，并返回数据
* 5、server响应，此次查询完成，连接关闭。

以下以设备信息查询指令`DeviceInformation`为例进一步分析每个过程。

### 查询设备信息的指令操作过程
* MDM Server 与 APNs建立连接，发送一个固定的指令,内容如下。

```
token=8c20addf006e09842376d9066fda4147800bc98755eb0430027a1a2f94442418 
payload=
{
"aps":
	{	
		"sound":"default.caf"
	},
"mdm":"EC0B1F96-5160-424C-A9DE-754A454E424B"
}
```
在这里需要我们前面得到的p12格式的证书，形式上和APP的差不多。其中token就是在`TokenUpdate`时的token，mdm是其中 的`PushMagic`，这个值是每次推送时都必须有的。所以根据内容看出Sever与APNs推送的消息基本固定，不同于APP的消息推送。发送这个消息主要目的就是通知设备，MDM Server要给你发指令了，赶快去连接服务器。

* 当设备收到APNs推送消息主动连接Server

收到有APNs发来的消息，发起请求到通过配置文件的服务器URL（即`ServerURL`字段的值）。向Server报告自己的当前状态是否空闲。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Status</key>
	<string>Idle</string>
	<key>UDID</key>
	<string>233deb277d03bd4aaf91108390c7d*</string>
</dict>
</plist>
```
以上可以看出每次请求或应答都会有UDID来标记设备，`Status`的值表示设备当前状态。状态值有以下几种状态：

| Status value  			| Description  	          |
| -----------------------	|-------------------------  |
| Acknowledged     		| 一切正常，设备正确响应指令    | 
| Error      				|       出现错误             |
| CommandFormatError 		|  指令格式错误              |  
|Idle						|	设备空闲		             |
|NotNow						|设备收到指令，但不能马上执行以后会再次请求服务器|

正常情况下大多数出现的是`Acknowledged` 和`Idle`两种状态。

* Server收到设备发来的状态信息

收到设备状态信息，判断是否空闲，只有空闲的时候再去发送指令。发送查询设备信息指令：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN""http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>Command</key>
		<dict>
			<key>RequestType</key>
			<string>DeviceInformation</string>
			<key>Queries</key>
			<array>
				<string>ModelName</string>
				<string>Model</string>
				<string>BatteryLevel</string>
				<string>DeviceCapacity</string>
				<string>AvailableDeviceCapacity</string>
				<string>OSVersion</string>
				<string>SerialNumber</string>
				<string>IMEI</string>
				<string>ICCID</string>
				<string>MEID</string>
				<string>IsSupervised</string>
				<string>IsDeviceLocatorServiceEnabled</string>
				<string>IsActivationLockEnabled</string>
				<string>IsCloudBackupEnabled</string>
				<string>WiFiMAC</string>
				<string>BluetoothMAC</string>
			</array>
		</dict>
		<key>CommandUUID</key>
		<string>f04997b8-aae2-44de-8c8d-8fb838000d0c</string>
	</dict>
</plist>
```
Server发送一个命令操作时必定包含`Command`和 `CommandUUID`

`Command`必须有`RequestType`表示具体的命令操作 + 该命令相关的操作参数。以上命令用来查询设备信息，`Queries`数组中表示要查询的内容的key。

`CommandUUID`表示命令的ID，当设备响应命令操作时，Sever可以此来确定是哪个命令操作，然后做相应的数据处理。


* 设备收到命令执行，根据指定的key返回相应的数据

```
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0"> 
	<dict> 
		<key>CommandUUID</key> 
		<string>f04997b8-aae2-44de-8c8d-8fb838000d0c</string> 
		<key>QueryResponses</key> 
		<dict> 
			<key>AvailableDeviceCapacity</key> 
			<real>19.606937408447266</real> 
			<key>BatteryLevel</key> 
			<real>0.56000000238418579</real> 
			<key>BluetoothMAC</key> 
			<string>6c:70:9f:2b:46:72</string> 
			<key>DeviceCapacity</key> 
			<real>26.413677215576172</real> 
			<key>ICCID</key> 
			<string>8986 0113 7231 0048 6168</string> 
			<key>IMEI</key> 
			<string>35 884805 093285 4</string> 
			<key>IsActivationLockEnabled</key> 
			<false /> 
			<key>IsCloudBackupEnabled</key> 
			<false /> 
			<key>IsDeviceLocatorServiceEnabled</key> 
			<false /> 
			<key>IsSupervised</key> 
			<false /> 
			<key>MEID</key> 
			<string>35884805093285</string> 
			<key>Model</key> 
			<string>ME824CH</string> 
			<key>ModelName</key> 
			<string>iPad</string> 
			<key>OSVersion</key> 
			<string>9.2.1</string> 
			<key>SerialNumber</key> 
			<string>F4KMG0FSFLMM</string> 
			<key>WiFiMAC</key> 
			<string>6c:70:9f:2b:46:71</string> 
		</dict> 
		<key>Status</key> 
		<string>Acknowledged</string> 
		<key>UDID</key> 
		<string>233deb277d03bd4aaf91108390c7d9fe2c49c8be</string> 
	</dict> 
</plist>

```


* server响应，若还需操作继续发送指令，否则返回为空此次操作完成，断开连接。

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Length: 0
Date: Wed, 26 Apr 2017 07:34:00 GMT
```

### 其他操作命令

* 查询设备已安装的应用

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN""http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0"><dict>
	<key>Command</key>
	<dict>
		<key>RequestType</key>
		<string>InstalledApplicationList</string>
	</dict>
	<key>CommandUUID</key>
	<string>149e4fd2-0267-4da2-9b58-bf94282dcdb4</string>
</dict>
</plist>
```


* 设备锁屏命令

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN""http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>Command</key>
		<dict>
			<key>RequestType</key>
			<string>DeviceLock</string>
		</dict>
		<key>CommandUUID</key>
		<string>07a6c20e-5e35-4f79-8680-10dee8460099</string>
	</dict>
</plist>
```

* 清除密码命令

```
     <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN"
     "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
     <plist version="1.0">
     <dict>
           <key>Command</key>
           <dict>
                 <key>RequestType</key>
                 <string>ClearPasscode</string>
                 <key>UnlockToken</key>
                 <data>
           			// base64编码的字符串（在TokenUpdate中获取的UnlockToken字段的值）
                 </data>
           </dict>
           <key>CommandUUID</key>
           <string></string>
     </dict>
     </plist>
```


###命令的请求和响应格式
* 命令请求格式

```
     <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN"
     "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
     <plist version="1.0">
     <dict>
           <key>Command</key>
           <dict>
                 <key>RequestType</key>
                 <string>命令名字</string>
                 ...其他字段或参数（可选），根据不同的命令会有不同的附加的key

           </dict>
           <key>CommandUUID</key>
           <string></string>
     </dict>
     </plist>
```

* 命令响应格式

```
<plist version="1.0">
     <dict>
         <key>CommandUUID</key>
         <string>CommandUUID</string>
         <key>Status</key>
         <string>Acknowledged</string>
         <key>UDID</key>
         <string>[device UUID]</string>
     </dict>
     </plist>
```

由设备发起的请求或响应操作基本是固定的，我们唯一能够操作的也只有Sever端的请求和响应了。

###参考：

1、MDM协议官方文档- **Mobile Device Management Protocol Reference** [https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/MobileDeviceManagementProtocolRef/3-MDM_Protocol/MDM_Protocol.html#//apple_ref/doc/uid/TP40017387-CH3-SW2](https://developer.apple.com/library/content/documentation/Miscellaneous/Reference/MobileDeviceManagementProtocolRef/3-MDM_Protocol/MDM_Protocol.html#//apple_ref/doc/uid/TP40017387-CH3-SW2)

2、配置描述文件参考- **Configuration Profile Reference**[https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010206-CH1-SW1](https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010206-CH1-SW1)