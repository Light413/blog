###简介
>
这个证书就是MDM Server 和 APNs推送消息所需要的证书，当然和APP推送证书完全不同，虽然功能差不多。

>MDM中分为Vendor 和Customer两个角色，即服务商和用户。
<br>如果你有个企业开发者账号，没错就是$299的那个，你既可以是Vendor,也可以是Customer。作为一个Vendor，你可以为Customer的证书请求文件颁发个用于签名的证书。

>如果你仅仅是个Customer也就是没有$299的账号，只能等Vendor给你颁发个`.cer`证书、签名产生个plist_encoded 文件，然后提交到[https://identity.apple.com/pushcert/](https://identity.apple.com/pushcert/)，如果文件不正确会提示格式错误，若正确会生成一个用于推送的证书`mdm.pem`。

以下以一个Vendor的角色进行以下操作。

### 1、 开通MDM服务功能

默认的企业开发者账号没有开通MDM服务,需要申请[开通MDM服务成为Vendor](https://developer.apple.com/contact/submit/)，输入企业账号和密码登录，提示你填写一些东西申请服务。如果你按要求填写了，提交了，然后就是傻傻的等待了。当时我提交后一周后也没反应，索性直接打电话人工客服，一分钟搞定。如果开通成功后会发邮箱通知，然后在制作证书的时候会出现MDM CSR选项。如下：

![MDM_CSR.png](http://upload-images.jianshu.io/upload_images/1859207-17a5ea2648d09fb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




### 2、生成MDM证书

####作为一个vendor首先要生成个用于签名的`.cer证书`，具体步骤如下

* 打开钥匙串生成`mdm_vendor.certSigningRequest`

* 导出秘钥`mdm_vendor.p12`记住导出密码下面签名时会用到

* 登录开发者中心制作`MDM CSR`类型的证书，下载即得到证书`mdm.cer`。此证书用来为customer生成的.csr证书文件签名。

####作为一个customer

* 终端中生成`customer.csr`文件
 
		openssl genrsa -des3 -out customerPrivateKey.pem 2048
		openssl req -new -key customerPrivateKey.pem -out customer.csr
	
	或者利用钥匙串生成，然后以.csr后缀保存，然后导出秘钥.p12格式并记住密码

* 提交`customer.csr`文件 给vendor 进行签名处理。

#### 对`customer.csr`签名
关于`customer.csr`文件的签名网上普遍有两种方法，一个Python脚本，一个时Java版的Softthinker,在此使用Python脚本。

[Python脚本源代码](https://github.com/grinich/mdmvendorsign)，使用过程中可能由于Mac中python版本有问题，无法下载所需的`AppleIncRootCertificate.cer`和`AppleWWDRCA.cer` 这两个官方提供的 证书。所以改了下脚本，直接把证书文件下载到本地。

根据要求需要把 `mdm_vendor.p12` 转化为 `mdm_vendor.key`格式的

然后在终端中执行以下操作

```
python mdm_vendor_sign.py 
--key mdm_vendor.key  
--csr customer.csr 
--mdm mdm.cer 
--root AppleIncRootCertificate.cer 
--WWDR AppleWWDRCA.cer 
```

执行过程中提示输入密码密码，结束，目录下多出一个"plist_encoded"的签名文件。
执行结果如图:

![MDM_SIGN.png](http://upload-images.jianshu.io/upload_images/1859207-832e7ff7b3e77683.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

——————

我的目录文件截图:

![MDM_RESULT.png](http://upload-images.jianshu.io/upload_images/1859207-c4ef8bc2441ec884.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[我用于签名的Python文件](https://github.com/Light413/images/tree/master/MDM_Python_Sign)为了方便操作在源文件的基础上做了些修改。

到[https://identity.apple.com/pushcert/](https://identity.apple.com/pushcert/) 提交 生成的`plist_encoded `文件，如果文件有问题会提示无效的文件，如果一切正常会生成我们最后需要的`MDM_Certificate.pem`的证书，以后server和APNs通信就是需这个证书。



### 3、验证MDM_Certificate.pem证书有效性并进行格式转化
以上得到了`MDM_Certificate.pem`,那么我们得到的这个是不是正确的呢？我们可以再终端中验证一下：

> `openssl s_client -connect gateway.push.apple.com:2195 -cert MDM_Certificate.pem -key customerPrivateKey.pem -debug -showcerts -status`

如果提示了错误，或连接直接closed则证书有问题。如果一直一直处于等待输入状态，输入任意、退出则证书是有效的。



接下来几乎网上所有的文章都是这样的

![image](https://github.com/Light413/blog/blob/master/img/MDM_Certificate.png?raw=true)

双击`MDM_Certificate.pem`安装，查看证书信息如图

![证书-用户ID](https://github.com/Light413/blog/blob/master/img/MDM_Certificate2.png?raw=true)

其中用户ID : `com.apple.mgmt.External.*` 这个很重要，在配置.mobileconfig文件要用到。


然后就没了，还有人说直接导出为.p12格式。但是为啥我的安装后根本导不出p12格式呢? 我们是Java后台所以需要p12格式的，这里就需要进一步的格式转化了。

在终端中:
>`openssl pkcs12 -export -in MDM_Certificate.pem -out MDM_Certificate.p12 -inkey customerPrivateKey.pem `


### 4、至此得到和APNs通信的证书`MDM_Certificate.p12`。

把证书和密码交给我们的后端人员，证书制作完成，是不是和APP的推送证书完全不同？我感觉完全是两个概念。


