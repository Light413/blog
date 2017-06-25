###简介
> 配置描述文件是一个用于安装到设备的XML格式的文件，包含了相关的配置信息。
比如：

>* 设备安全性策略和访问限制
* VPN 配置信息
* Wi-Fi 设置
* 密码策略设置
* 移动设备管理
* 邮件和日历帐户等

制作一个配置文件可以`iPhone配置使用工具`和`手写XML文件`两种方式。为了方便操作我用了前者（网上说这个已被苹果抛弃，好像不影响文件生成）。

###用iPhone配置使用工具生成配置文件
打开文件——新建配置文件，主要涉及使用到的配置如下：

![Snip20170502_3.png](http://upload-images.jianshu.io/upload_images/1859207-d219c7dfdd7a1765.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 通用：设置配置文件的基本信息，其中`标示符`和APP ID类似，`安全性`表示是否可移除该描述文件默认`总是`允许删除、可选择`永不：表示禁止删除，使用授权：表示需要密码验证`。


* 限制：设置设备访问权限，比如是否允许安装应用、是否允许相机、iCloud等，按默认设置即可。


* 凭证：用于和MDM Server 认证的一个p12的证书文件。

* 移动设备管理：这一步配置尤为关键，设置如下。

![Snip20170502_4.png](http://upload-images.jianshu.io/upload_images/1859207-3207b4b4a3f53087.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> * 服务器URL：设备注册后以后每次连接的URL地址。
> * 登记URL：首次设备注册的地址，包括认证和更新token操作。
> * 主题：在上一篇证书制作中提到的 用户ID : com.apple.mgmt.External.*。
> * 身份：在凭证中添加的证书。
> * 移除时检查：当为TRUE，当用户删除设备上的配置文件时设备会向登记URL发送个消息表示配置文件要删除了，MDM Serve可以依此来检测设备是否还在监控中。
> * 访问权限：按默认即可。
> * Apple 推送通知服务：选中表示使用的开发环境APNs,这里不要选中。

至此配置设置基本完成，保存、导出会提示给配置文件签名，选择无即可。

###生成的完整的XML文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>PayloadContent</key>
	<array>
		<dict>
			<key>PayloadDescription</key>
			<string>配置访问限制</string>
			<key>PayloadDisplayName</key>
			<string>访问限制</string>
			<key>PayloadIdentifier</key>
			<string>com.apple.applicationaccess.C6130962-2621-47FD-8E9C-8832BCE3C5B0</string>
			<key>PayloadType</key>
			<string>com.apple.applicationaccess</string>
			<key>PayloadUUID</key>
			<string>C6130962-2621-47FD-8E9C-8832BCE3C5B0</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>allowActivityContinuation</key>
			<true/>
			<key>allowAddingGameCenterFriends</key>
			<true/>
			<key>allowAppCellularDataModification</key>
			<true/>
			<key>allowAppInstallation</key>
			<true/>
			<key>allowAppRemoval</key>
			<true/>
			<key>allowAssistant</key>
			<true/>
			<key>allowAssistantWhileLocked</key>
			<true/>
			<key>allowAutoCorrection</key>
			<true/>
			<key>allowAutomaticAppDownloads</key>
			<true/>
			<key>allowBluetoothModification</key>
			<true/>
			<key>allowBookstore</key>
			<true/>
			<key>allowBookstoreErotica</key>
			<true/>
			<key>allowCamera</key>
			<true/>
			<key>allowChat</key>
			<true/>
			<key>allowCloudBackup</key>
			<true/>
			<key>allowCloudDocumentSync</key>
			<true/>
			<key>allowCloudPhotoLibrary</key>
			<true/>
			<key>allowDefinitionLookup</key>
			<true/>
			<key>allowDeviceNameModification</key>
			<true/>
			<key>allowEnablingRestrictions</key>
			<true/>
			<key>allowEnterpriseAppTrust</key>
			<true/>
			<key>allowEnterpriseBookBackup</key>
			<true/>
			<key>allowEnterpriseBookMetadataSync</key>
			<true/>
			<key>allowEraseContentAndSettings</key>
			<true/>
			<key>allowExplicitContent</key>
			<true/>
			<key>allowFingerprintForUnlock</key>
			<true/>
			<key>allowFingerprintModification</key>
			<true/>
			<key>allowGameCenter</key>
			<true/>
			<key>allowGlobalBackgroundFetchWhenRoaming</key>
			<true/>
			<key>allowInAppPurchases</key>
			<true/>
			<key>allowKeyboardShortcuts</key>
			<true/>
			<key>allowManagedAppsCloudSync</key>
			<true/>
			<key>allowMultiplayerGaming</key>
			<true/>
			<key>allowMusicService</key>
			<true/>
			<key>allowNews</key>
			<true/>
			<key>allowNotificationsModification</key>
			<true/>
			<key>allowOpenFromManagedToUnmanaged</key>
			<true/>
			<key>allowOpenFromUnmanagedToManaged</key>
			<true/>
			<key>allowPairedWatch</key>
			<true/>
			<key>allowPassbookWhileLocked</key>
			<true/>
			<key>allowPasscodeModification</key>
			<true/>
			<key>allowPhotoStream</key>
			<true/>
			<key>allowPredictiveKeyboard</key>
			<true/>
			<key>allowRadioService</key>
			<true/>
			<key>allowRemoteScreenObservation</key>
			<true/>
			<key>allowSafari</key>
			<true/>
			<key>allowScreenShot</key>
			<true/>
			<key>allowSharedStream</key>
			<true/>
			<key>allowSpellCheck</key>
			<true/>
			<key>allowSpotlightInternetResults</key>
			<true/>
			<key>allowUIAppInstallation</key>
			<true/>
			<key>allowUIConfigurationProfileInstallation</key>
			<true/>
			<key>allowUntrustedTLSPrompt</key>
			<true/>
			<key>allowVideoConferencing</key>
			<true/>
			<key>allowVoiceDialing</key>
			<true/>
			<key>allowWallpaperModification</key>
			<true/>
			<key>allowiTunes</key>
			<true/>
			<key>forceAirDropUnmanaged</key>
			<false/>
			<key>forceAssistantProfanityFilter</key>
			<false/>
			<key>forceEncryptedBackup</key>
			<false/>
			<key>forceITunesStorePasswordEntry</key>
			<false/>
			<key>forceWatchWristDetection</key>
			<false/>
			<key>ratingApps</key>
			<integer>1000</integer>
			<key>ratingMovies</key>
			<integer>1000</integer>
			<key>ratingRegion</key>
			<string>us</string>
			<key>ratingTVShows</key>
			<integer>1000</integer>
			<key>safariAcceptCookies</key>
			<integer>2</integer>
			<key>safariAllowAutoFill</key>
			<true/>
			<key>safariAllowJavaScript</key>
			<true/>
			<key>safariAllowPopups</key>
			<true/>
			<key>safariForceFraudWarning</key>
			<false/>
		</dict>
		<dict>
			<key>PayloadDescription</key>
			<string>配置密码设置</string>
			<key>PayloadDisplayName</key>
			<string>密码</string>
			<key>PayloadIdentifier</key>
			<string>com.apple.mobiledevice.passwordpolicy.B52AEECB-63DD-4B05-AFB2-6B547038F8D7</string>
			<key>PayloadType</key>
			<string>com.apple.mobiledevice.passwordpolicy</string>
			<key>PayloadUUID</key>
			<string>B52AEECB-63DD-4B05-AFB2-6B547038F8D7</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>allowSimple</key>
			<true/>
			<key>forcePIN</key>
			<true/>
			<key>requireAlphanumeric</key>
			<false/>
		</dict>
		<dict>
			<key>Password</key>
			<string>123456</string>
			<key>PayloadCertificateFileName</key>
			<string>out.p12</string>
			<key>PayloadContent</key>
			<data>
			//证书内容base64编码的字符串
			</data>
			<key>PayloadDescription</key>
			<string>提供设备鉴定（证书或身份）。</string>
			<key>PayloadDisplayName</key>
			<string>out.p12</string>
			<key>PayloadIdentifier</key>
			<string>com.apple.security.pkcs12.ACACFDA4-64B4-46D5-A8BD-DB241775A394</string>
			<key>PayloadOrganization</key>
			<string>Gener-Tech</string>
			<key>PayloadType</key>
			<string>com.apple.security.pkcs12</string>
			<key>PayloadUUID</key>
			<string>ACACFDA4-64B4-46D5-A8BD-DB241775A394</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
		</dict>
		<dict>
			<key>AccessRights</key>
			<integer>8191</integer>
			<key>CheckInURL</key>
			<string>https://www..../checkin.do?deviceId=fc97f6b4524346a18f14d1a425986abb</string>
			<key>CheckOutWhenRemoved</key>
			<true/>
			<key>IdentityCertificateUUID</key>
			<string>ACACFDA4-64B4-46D5-A8BD-DB241775A394</string>
			<key>PayloadDescription</key>
			<string>配置“移动设备管理”</string>
			<key>PayloadDisplayName</key>
			<string>移动设备管理</string>
			<key>PayloadIdentifier</key>
			<string>com.apple.mdm.02D2C93A-3F6D-4E54-B15D-EECC1B7BD583</string>
			<key>PayloadOrganization</key>
			<string>Gener-Tech</string>
			<key>PayloadType</key>
			<string>com.apple.mdm</string>
			<key>PayloadUUID</key>
			<string>02D2C93A-3F6D-4E54-B15D-EECC1B7BD583</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>ServerURL</key>
			<string>https://www..../mdm/server.do?deviceId=fc97f6b4524346a18f14d1a425986abb</string>
			<key>SignMessage</key>
			<true/>
			<key>Topic</key>
			<string>com.apple.mgmt.External.*</string>
			<key>UseDevelopmentAPNS</key>
			<true/>
		</dict>
	</array>
	<key>PayloadDescription</key>
	<string>Lock&amp;Reset All Settings&amp;Erase</string>
	<key>PayloadDisplayName</key>
	<string>Gener MDM Sever</string>
	<key>PayloadIdentifier</key>
	<string>net.myfleet.mdm</string>
	<key>PayloadOrganization</key>
	<string>Gener-Tech</string>
	<key>PayloadRemovalDisallowed</key>
	<false/>
	<key>PayloadType</key>
	<string>Configuration</string>
	<key>PayloadUUID</key>
	<string>984CE2FF-6BE1-49AE-A3EF-43B0B0EC9A11</string>
	<key>PayloadVersion</key>
	<integer>1</integer>
</dict>
</plist>


```

我们可以直接修改此XML文件，据此[Configuration Profile Reference](https://developer.apple.com/library/content/featuredarticles/iPhoneConfigurationProfileRef/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010206-CH1-SW1) 可添加其他所需的字段。你也可以在此基础上修改适合为自己的（估计很容易遗漏或出错），我还是喜欢在`iPhone配置使用工具`中操作比较方便。


###给生成的配置文件签名
以上生成的配置文件其实可以直接安装到设备上，如果安装成功后会有一个红色的提示‘未签名’如下。<br>
![Snip20170508_1.png](http://upload-images.jianshu.io/upload_images/1859207-e005c7daaf93a3bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

签名要经历两个操作，一、MDM Sever端签名。二、用苹果颁发的证书签名。

#####MDM Sever签名
需要以下证书文件：

* unsigned.mobileconfig 原始的未签过名的配置文件
* server.crt 服务器端用于签名的证书
* server.key 服务器端用于签名的证书的秘钥
* cert-chain.crt 其他机构为服务器颁发的CA证书
* signed.mobileconfig will be your signed configuration profile

可以再终端中执行：`openssl smime -sign -in unsigned.mobileconfig -out signed.mobileconfig -signer server.crt -inkey server.key -certfile cert-chain.crt -outform der -nodetach`

(以上是Java后台签名的操作过程，我没有验证，在此作为一个操作步骤总结放在这里)。

我猜测MDM Sever的签名只是为了和客户端进行下认证和对描述文件的加密过程，只是让这两个之间相互认知对方，和iOS系统是否承认无关。所以以上操作之后还会提示‘未签名’。（实际测试中这个操作可以省略）。


#####苹果证书相关的签名
以下操作引自 天狐博客，文章地址[http://www.skyfox.org/ios-mobileconfig-sign.html](http://www.skyfox.org/ios-mobileconfig-sign.html)

这个操作有几种方法可供选择，这里我使用了脚本签名。

借助于强大的github,找到了一个python脚本进行签名

地址:[https://github.com/nmcspadden/ProfileSigner](https://github.com/nmcspadden/ProfileSigner)

1.签名一个mobileconfig

profile_signer.py与 mobileconfig 放在同一目录,终端进入目录执行

 `./profile_signer.py -n "3rd Party Mac Developer Application" sign AcrobatPro.mobileconfig AcrobatProSigned.mobileconfig`
2.加密一个mobileconfig

`
 ./profile_signer.py -n "3rd Party Mac Developer Application" encrypt AcrobatPro.mobileconfig AcrobatProEnc.mobileconfig`
3.签名并且加密一个mobileconfig

`
 ./profile_signer.py -n "3rd Party Mac Developer Application" both AcrobatPro.mobileconfig AcrobatProBoth.mobileconfig`
"3rd Party Mac Developer Application"为你的证书在钥匙串中的全名,选择证书=>显示简介=>复制常用名称加上引号即可,比如

"iPhone Developer: jakey.shao xxxx@xxx.com"

"iPhone Distribution: Skyfox Network Technology Co., Ltd."

66911171-EE9C-4DB7-BFCE-6564CC1B4E1A如果能正确读取到证书，会提示允许访问钥匙串，点击允许即可！

最终安装提示已验证啦。

![Snip20170508_2.png](http://upload-images.jianshu.io/upload_images/1859207-db213e051ca194a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 至此得到mobileconfig配置文件，交由MDM Sever供设备下载。







