

# 使用Sparkle实现软件的检查更新

上sparkle官方连接:https://sparkle-project.org/ 闪闪放光的sparkle是开源的，它非常好的实现了软件的更新功能。

不像有些软件，检查更新只提供一个下载链接，让用户去下载，下载完成之后再进行安装更新。sparkle可以直接进行软件的安装和更新操作。



>  当然了，这里提到的更新都是App Store之外的应用更新，上架App Store的应用不允许提供检查更新和更新的功能，这些功能由App Store负责，App Store的更新不在此讨论范围之内。



### 1.在项目中集成sparkle

我这里使用cocoapod进行集成，当然也可以手动集成

```sh
pod 'Sparkle'
```



### 2.添加ED公钥

在info.plist文件中添加ED公钥，

key为：SUPublicEDKey  

value: 获取的公钥

如何获取公钥呢？

运行 ./bin/generate_keys 可以获取公钥，generate_keys 这个文件是Sparkle工具提供的，

可以在这里下载最新的版本:https://github.com/sparkle-project/Sparkle/releases/

我下载的版本是1.2.3.0,无需下载源码，下载编译好的工具即可，我下载的是[Sparkle-1.23.0.tar.xz](https://github.com/sparkle-project/Sparkle/releases/download/1.23.0/Sparkle-1.23.0.tar.xz)

generate_keys 就在解压之后的/bin/目录下,这个/bin目录下还有用于签名的sign_update工具

执行./bin/generate_keys 可以获取公钥，如果没有，就再执行一次。

### 3.添加用于解析更新的xml文件地址

在info.plist文件添加此内容

key:SUFeedURL

value: 用于解析更新内容的xml地址

这里要注意两点：

- 尽可能使用https://地址，而不是http://，如果使用http,在额外需要在info.plist文件中添加ATS允许加载http
- xml要可以直接获取到原始文件xml文件到地址，比如，如果xml文件放在了github上，我测试的时候就是这么玩的，我使用的地址是https://github.com/MyColourfulLife/Update/blob/master/SampleAppcast.xml 然而这个地址是不可以的，在其他地方或许可以可以，返回的直接就是xml文件本身，而github返回的则是包含xml文件的网页，这对sparkle解析起来不仅慢而且会出问题。在github上，正确的地址应该是https://raw.githubusercontent.com/MyColourfulLife/Update/master/SampleAppcast.xml 这样才是正确的结果。

至于xml文件的内容，这个也是重中之重。



### 4.编写xml文件

上面的连接已经给出了示例，这里主要说明一下

更新内容的连接在releaseNotesLink标签里，

```xml
<sparkle:releaseNotesLink>
                http://you.com/app/2.0.html
            </sparkle:releaseNotesLink>
```

也可以直接使用下面的方式，直接说明更新内容

```xml
<description>
          <![CDATA[
            <ul>
              <li>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</li>
              <li>Suspendisse sed felis ac ante ultrices rhoncus. Etiam quis elit vel nibh placerat facilisis in id leo.</li>
              <li>Vestibulum nec tortor odio, nec malesuada libero. Cras vel convallis nunc.</li>
              <li>Suspendisse tristique massa eget velit consequat tincidunt. Praesent sodales hendrerit <a href="#">pretium</a>.</li>
            </ul>
          ]]>
        </description>
```

更新日期

```xml
 <pubDate>Wed, 09 April 2020 19:20:11 +0000</pubDate>
```

版本-下载地址-文件大小-版本号 在下面的标签里

```xml
<enclosure url="http://0.0.0.0:8000/Update.app.zip" sparkle:version="2.0" length="4945096" type="application/octet-stream" sparkle:edSignature="8jomTVaWC7p3rxgNW8xXlRK2oe4eyFQiJ9+oknCIoKrH8+sF7jy2xQ+Mx37F00WenDBuOIy2ymD3nTQuQkyOCw==" />
```

可以设置最低的系统版本支持

```xml
<sparkle:minimumSystemVersion>10.13</sparkle:minimumSystemVersion>
```

## 5.在软件中添加Sparkle对象

1. 打开main.storyboard
2. 在资源库中添加Object对象到Application场景目录下，并修改Object的class为SUUpdater

![image-20200410123002060](https://upload-images.jianshu.io/upload_images/2659589-bb3e39ba81479c1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

3. 添加一个检查更新的菜单到主菜单，将检查更新菜单拖拽到刚才的Updater对象上(就是刚才的Object)，绑定Updater的 checkForUpdate事件，即可。
