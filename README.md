阿里云oss sts签名直传demo  
=

#### 此demo是对基于post policy签名方式的改善方案  
post policy签名方式的介绍:  
https://help.aliyun.com/document_detail/31926.html  


#### 阿里云提供的基于post policy的签名方式demo有什么问题:  
官方demo依赖一对静态的accesskey accesskeysecret,在一个有安全规则的企业环境中这种credentials
是需要定期跟换的,并且这些accesskey accesskeysecret一般不明文存放.

#### 此demo怎么解决这个问题(工作原理):  
1. 将操作oss的role授权给运行签名服务的ecs服务器,这样签名程序可以使用ecs的metadata获取到临时的sts credential包含accesskey,accesskeysecret,security_token,不用配置静态的accesskey 到代码里面.解决静态配置的问题,也解决直接看得到凭证明文的问题 
2. 签名程序还是按照正常的签名逻辑运行,使用从metadata url里面获取到的accesskeysecret去签名signature,最后生成签过名的signature  
3. 由于是使用sts token里面的 accesskeysecret签名的signature,因此要多传一个`x-oss-security-token`参数到最终的params里面

#### 此demo的限制:
- 只能在阿里云环境内运行,因为使用了instance role的授权方式.而官方的demo可以在阿里云环境以外运行.
- 由于在内网环境测试,去掉了callback相关配置,因为oss不允许内网回调地址.有需要的可以将回调地址暴露到公网.

如何运行这个demo
=
#### 前提要求:
- python3
- 预先在阿里云ram服务创建一个权限为`AliyunOSSFullAccess`的role授权给签名的ecs服务器(**生产环境不要给AliyunOSSFullAccess权限,应该设置更细的策略限制到具体的bucket和action**)

#### running:
安装requests库:
```bash
python3 -m pip install requests

```
**修改服务端和客户端脚本适配自己的环境:**  
服务端: oss-sign-python/appserver.py
```bash
bucket_name = 'bucketname' #mybucket
region = 'region' #cn-hangzhou
host = f'https://{bucket_name}.oss-{region}.aliyuncs.com';
rolename = "rolename" #myoss-demo-role
upload_dir = 'user-dir-prefix/'
expire_time = 30
```
客户端:  
oss-web-client/upload.js
```javascript
serverUrl = 'http://签名服务器ip地址:9097'
```
**启动签名服务端/客户端:**
```bash
cd oss-sign-python
python3 appserver.py 9097 &

cd oss-web-client
python3 -m http.server 9098 &
```
**测试**:
浏览器打开http://客户端ip:9098 ,选取一个txt文件或者图片文件上传上去.

