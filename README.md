# TVBoxOSC
### 影视解析原理
```
1. 寻找站源网站
2. 爬取站源网站内容，分析获取播放地址
3. 寻找解析网站
4. 将获取的播放地址传给解析网站接口达到能播放的效果

1. 资源站
  在www上找到相对稳定的站源，站源的意思是影视资源的网站（含官方和非官方），没经验可以自己搭建一个影视CMS网站了解（十多年的开源项目，很成熟，如：苹果CMS，赤兔CMS，英皇CMS...）
 ；
 一般站源都有对应的接口用于提供给第三方采集，比如一般访问网站地址www.xxx.com，而专门的采集接口是www.xxx.com/api.php/v1.vod或者www.xxx.com/api.php/provide/vod/at/xml；
 采集接口一般有一个对应的规则接口，可以理解成网站内容说明书，可以参考来分析采集回来的数据；
 采集回来的数据是含有视频地址的，如https://www.xxx.com/uploads/lxj/mhl/01.m3u8
2. 解析站 
  站源的视频地址一般是无法直接点开播放的，因为资源可能涉及到会员，加密...所以还需要依赖第三方的解析网站实现解密之类的过程；
  相对稳定的解析站也是去常用的影视CMS网站了解吧；
  使用方式很简单：
  1. 解析站地址 http://www.xxx.com/home/api?type=ys&uid=188175&key=aabbccddeeff654321&url=
  2. 填入要解析的资源地址
    如：http://www.xxx.com/home/api?type=ys&uid=188175&key=aabbccddeeff654321&url=https://www.xxx.com/uploads/lxj/mhl/01.m3u8
  3. 在浏览器输入或者播放器填入拼接好的地址就可以播放了

以上难点，如何找到稳定的资源站和解析站，这基本都是人工去找，这类网站很容易被封，但是又很容易复活，其本质就是开源的影视CMS，建站成本很低，都是一套逻辑，所以新站的采集等接口都是一样的
就是改域名的问题（不过现在有些大的站源慢慢走封闭的路线了，付费采集...），这也就是为什么这类影视APP经常出现无法解析/播放的原因。

3. IPTV电视直播
  github上找到官方的源（关键词：iptv），如：https://www.cctv.com/cctv1.m3u8 用vlc之类的播放器就能播放，没有加密
```

### APP原理（解析+展示+播放）
```
1. 人工整理多个站源和解析站，编辑成一个json文件
2. APP解析json文件，然后请求采集站数据展示在APP
3. 点击播放，APP拼接成解析站需要的链接，然后使用IJK播放器播放，完成

1.0 最简单粗暴的开发方式是：把解析的代码逻辑和站源json文件都打包进apk，当站源或者解析站失效需要修复时，用户需要更新整个APP；

2.0 进阶的开发方式是：把站源json文件做成外部加载的方式，简单理解成网盘地址，这样当站源失效修改网盘json即可（CMS建站成本低，大多换换域名而已），用户不需要更新APP，
但是有如果新的站源需要重新开发解析逻辑，那么用户还是需要更新APP才能使用；

3.0 高阶的开发方式是：解析的代码逻辑和站源json文件都不打包进apk，apk就是个空壳软件，解析的代码逻辑打包成一个外部的jar，当APP启动时动态下载远端的站源json文件和远端的jar包的本机加载，
从而实现对用户无感更新（APP展示逻辑都一样不需要变化：封面，简介，播放地址）；

4.0 自己搭建CMS管理提供站源数据 + 自己定制实现APP

apk加载外部jar用到的原理：java多态(接口编程)和类加载机制，apk定义通用接口IUser类，jar实现接口IUser的类Man，然后通过反射机制实现调用。
DexClassLoader classLoader = new DexClassLoader(cache, "/sdcard/data/xxxx.jar", null, App.getInstance().getClassLoader());
Class clazz = classLoader.loadClass("com.github.Man");
Method method = classInit.getMethod("sayHi", String.class);
method.invoke(clazz.newInstance(), "您吃了吗?");

TVBOX是在高阶方案基础上的优化网络问题，即增加了缓存站源json文件和jar到本机的功能，这样当网盘的文件被封时还能继续用。
实现方案是用natohttp框架在apk搭建一个web服务，用来实现文件的上传管理，然后也是通过该服务实现对存储空间的文件访问。
1. 通过http协议将xxx.json和xxx.jar上传到手机的/sdcard/data/目录，即/sdcard/tvconfig/xxx.json, /sdcard/tvconfig/xxx.jar
2. 通过http协议假装访问本地的xxx.json和xxx.jar，模拟实现把两者下载到本地加载的过程，http://127.0.0.1:8899/file/tvconfig/xxx.json, http://127.0.0.1:8899/file/tvconfig/xxx.jar （实际需不需下载有逻辑判断是否走缓存，另外要加一个路径要加file/，否则404，不懂为啥）
  为什么要假装访问本地的xxx.json和xxx.jar，直接访问本地存储路径不就行了吗？为了同时兼容远端加载和本地加载的逻辑，都是一套代码，做个前缀匹配替换更方便。
  http://172.10.10.10:8080/config/xxx.json vs clan://localhost/config/xxx.json(会替换成http://127.0.0.1:8899/file/tvconfig/xxx.json 相当于也是访问远端资源) 之后的逻辑就都一样了。

另外的优化就是适当实现一些多线程异步操作，让APP使用更流畅
```






















