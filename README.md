![](https://upload-images.jianshu.io/upload_images/6545546-d47d3abfd63cb1d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#前言
>项目上线之际总需要打包测试，每次使用`Xcode - Archive `等待好几分钟的Buliding，然后Export，通过`Application Loader`上传到 `AppStore `，或者分发到`蒲公英，Fir`等发布平台。一系列操作是耗时且没有技术含量的工作，所以自动化打包ipa，很有必要掌握。[点击前往Github下载Demo](https://github.com/wnkpzzz/CZAutoPackaging)
 





#工具
>脚本工具有很多，这里使用的是 `FastLane `，[Github](https://github.com/fastlane/fastlane)里面24+K的Star和3.7K的Fork，说明大量的开发者信任并一起维护使用。

#安装
使用 `FastLane `需要以下内容：
>* `OS X10.9`及以上版本
>* ` Ruby2.0`及以上版本
>* Xcode命令行工具`（CLT）`
>* 付费Apple开发者帐户

`FastLane`是`Ruby`脚本的集合，首先需要正确版本的`Ruby`,从`OS X10.9`及以后默认`Ruby2.0`。
我们可以通过命令确认，打开终端输入如下命令：
```
ruby -v
```
检测是否安装了 `Xcode CLT`  打开终端输入如下命令：
```
xcode-select --install
```
如果已经安装了`Xcode CLT` ，则会收到
```
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```
如果未安装，它将为您安装`Xcode CLT`。
完成先决条件后，您就可以安装`FastLane`了。 输入以下命令：
```
sudo gem install -n /usr/local/bin fastlane --verbose
```
正常情况，输入系统密码之后，终端窗口会有一系列活动，表明安装正在进行中，需要等待几分钟。
`备注：这里如果报错，参考文章最下面 踩坑一`

#配置FastLane
###1、fastlane init
新建一个项目工程，名字暂定 `fastlaneTest`，cd 到项目根目录执行
```
fastlane init
```
这里会弹出四个选项，问你想要用Fastlane做什么？ 这里我选的是3
```
1. 📸  Automate screenshots
2. 👩‍✈️  Automate beta distribution to TestFlight (自动testfilght型配置)
3. 🚀  Automate App Store distribution (自动发布型配置)
4. 🛠  Manual setup - manually setup your project to automate your (需要手动配置内容)
```
>如果你的工程是用cocoapods的那么可能会提示让你勾选工程的Scheme；
步骤就是打开你的xcode，
点击Manage Schemes，
找到你的项目Scheme，在后面的多选框中进行勾选，
然后可以手动删除 fastlane文件夹，重新fastlane init一下。
 ###2、登录Apple ID  
```
[11:10:04]: ----------------------------------------------------------
[11:10:04]: --- Setting up fastlane for iOS App Store distribution ---
[11:10:04]: ----------------------------------------------------------
[11:10:04]: Parsing your local Xcode project to find the available schemes and the app identifier
[11:10:05]: $ xcodebuild -showBuildSettings -scheme fastlaneTest -project fastlaneTest.xcodeproj
[11:10:06]: $ cd /Users/hans3d/Desktop/fastlaneTest && agvtool what-version -terse
[11:10:07]: --------------------------------
[11:10:07]: --- Login with your Apple ID ---
[11:10:07]: --------------------------------
[11:10:07]: To use App Store Connect and Apple Developer Portal features as part of fastlane,
[11:10:07]: we will ask you for your Apple ID username and password
[11:10:07]: This is necessary for certain fastlane features, for example:
[11:10:07]: 
[11:10:07]: - Create and manage your provisioning profiles on the Developer Portal
[11:10:07]: - Upload and manage TestFlight and App Store builds on App Store Connect
[11:10:07]: - Manage your App Store Connect app metadata and screenshots
[11:10:07]: 
[11:10:07]: Your Apple ID credentials will only be stored in your Keychain, on your local machine
[11:10:07]: For more information, check out
[11:10:07]: 	https://github.com/fastlane/fastlane/tree/master/credentials_manager
[11:10:07]: 
[11:10:07]: Please enter your Apple ID developer credentials
[11:10:07]: Apple ID Username:
```
输入`Apple Developer`的账号和密码，因为2019.03苹果开启二级认证，需要手机登录开发者账号，进行允许权限。
登陆成功如下：
```
[11:41:06]: ✅  Logging in with your Apple ID was successful
[11:41:06]: Checking if the app 'com.cz.****.fastlaneTest' exists in your Apple Developer Portal...
[11:41:07]: It looks like the app 'com.cz.****.fastlaneTest' isn't available on the Apple Developer Portal
[11:41:07]: for the team ID '9A77SS**JG' on Apple ID '****@163.com '
[11:41:07]: Do you want fastlane to create the App ID for you on the Apple Developer Portal? (y/n)
```
选择 y
```
+----------------+--------------------------+
|        Summary for produce 2.118.1        |
+----------------+--------------------------+
| username       | ****@163.com           |
| team_id        | 9A77SS**JG               |
| itc_team_id    | 118370***                |
| platform       | ios                      |
| app_identifier | com.cz.****.fastlaneTest |
| skip_itc       | true                     |
| sku            | 1552967***               |
| language       | English                  |
| skip_devcenter | false                    |
+----------------+--------------------------+
```
然后会提示输入App Name：
```
[11:43:46]: App Name: fastlane_wnkp_demo
```
然后根据操作点击回车即可！此时目录结构大概如下所示：

![](https://upload-images.jianshu.io/upload_images/6545546-55237d830ff45dfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>`Appfile`，用于存储应用程序标识符和Apple ID。
`Fastfile`，用于管理您创建的用于调用某些操作的通道。最主要的文件，在这个文件中可以编写我们需要使用的各个工具的顺序、方式等。

 ###3、配置脚本

通过Xcode打开`Fastfile`文件如下
```
default_platform(:ios)

platform :ios do
  desc "Push a new release build to the App Store"
  lane :release do
    build_app(scheme: "fastlaneTest")
    upload_to_app_store(skip_metadata: true, skip_screenshots: true)
  end
end
```
编写内容代码如下：
```
default_platform(:ios)

platform :ios do

    desc "fastlane 打包上传到蒲公英发布网站"
    lane :fsv_pgyer do

    gym(
        clean:true,                      # Clean项目
        scheme:"fastlaneTest",           # 项目名称
        export_method:"ad-hoc",          # 打包的类型
        configuration:"Release",         # 模式，默认Release，还有Debug
        output_directory:"./build",      # 输出的位置
    )
end
end

```
>一个lane就是一个任务，`fsv_pgyer` 名字可以自己改。
`gym`是fastlane提供的打包工具,括号里面就是你自己配置的一些东西。[gym科普](https://docs.fastlane.tools/actions/gym/)

 ###4、执行命令、本地打包
最后就是打开终端，在你的工程目录下，运行` fastlane fsv_pgyer `就行了。
```
fastlane fsv_pgyer 
```
编译成功：
```
[15:49:55]: Successfully exported and compressed dSYM file
[15:49:55]: Successfully exported and signed the ipa file:
[15:49:55]: /Users/hans3d/Desktop/fastlaneTest/build/fastlaneTest.ipa

+------+------------------+-------------+
|           fastlane summary            |
+------+------------------+-------------+
| Step | Action           | Time (in s) |
+------+------------------+-------------+
| 1    | default_platform | 0           |
| 2    | gym              | 12          |
+------+------------------+-------------+

[15:49:55]: fastlane.tools finished successfully 🎉
```
打开项目目录，会发现有一个bulid文件夹，打开即是我们的ipa包
![](https://upload-images.jianshu.io/upload_images/6545546-e68f3e1315bab6d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ###5、上传到蒲公英或者fir配置
1、cd到项目下，安装pgyer插件执行命令 、
```
 fastlane add_plugin pgyer
```
执行结果：
```
[15:55:52]: Plugin 'fastlane-plugin-pgyer' was added to './fastlane/Pluginfile'
[15:55:52]: It looks like fastlane plugins are not yet set up for this project.
[15:55:52]: fastlane will modify your existing Gemfile at path '/Users/hans3d/Desktop/fastlaneTest/Gemfile'
[15:55:52]: This change is necessary for fastlane plugins to work
[15:55:52]: Should fastlane modify the Gemfile at path '/Users/hans3d/Desktop/fastlaneTest/Gemfile' for you? (y/n)
y
[15:55:59]: Successfully modified '/Users/hans3d/Desktop/fastlaneTest/Gemfile'
[15:55:59]: Make sure to commit your Gemfile, Gemfile.lock and Pluginfile to version control
Installing plugin dependencies...
Successfully installed plugins
```
2、重新编写项目目录下的Fastfile文件，如下：
```
default_platform(:ios)

platform :ios do

    desc "fastlane 打包上传到蒲公英发布网站"
    lane :fsv_pgyer do

    gym(
        clean:true,                      # Clean项目
        scheme:"fastlaneTest",           # 项目名称
        export_method:"ad-hoc",          # 打包的类型
        configuration:"Release",         # 模式，默认Release，还有Debug
        output_directory:"./build",      # 输出的位置
    )

    time      = Time.new.strftime("%Y%m%d")
    version   = get_version_number
    api_key   = "06e5563d5a8b94aac1c297426d351df6"
    user_key  = "974d1feb99ad49c955ed82a16ae2c21a"
    usipaName = "Release_#{version}_#{time}.ipa"

    pgyer(api_key: "#{api_key}", user_key: "#{user_key}")

end
end

```

>pgyer中的`api_key`和`user_key`为蒲公英的应用->API中查看，如下图：
蒲公英配置 [蒲公英文档](https://www.pgyer.com/doc/view/fastlane)

![](https://upload-images.jianshu.io/upload_images/6545546-32021542059952ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、cd到项目根目录下，执行命令：
```
fastlane fsv_pgyer 
```
输出结果：
```
[16:03:23]: Upload success. Visit this URL to see: https://www.pgyer.com/jnfC

+------+--------------------+-------------+
|            fastlane summary             |
+------+--------------------+-------------+
| Step | Action             | Time (in s) |
+------+--------------------+-------------+
| 1    | default_platform   | 0           |
| 2    | gym                | 12          |
| 3    | get_version_number | 0           |
| 4    | pgyer              | 1           |
+------+--------------------+-------------+
[16:03:23]: fastlane.tools finished successfully 🎉
```
打包上传成功，收到短信：
![](https://upload-images.jianshu.io/upload_images/6545546-ebbd4efd0bcb7e76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
-------------
#踩坑一
执行命令`sudo gem install -n /usr/local/bin fastlane --verbose`之后如下提示：
```
Could not find a valid gem 'fastlane' (>= 0), here is why:
          Unable to download data from https://ruby.taobao.org - SSL_connect returned=1 errno=0 state=error: certificate verify failed (https://ruby.taobao.org/specs.4.8.gz)
```
错误原因：
查看`gem`源
```
gem sources
```
输出结果：
```
*** CURRENT SOURCES ***
https://ruby.taobao.org
```
#####所以需要更换源
删除之前的源
```
gem sources --remove https://ruby.taobao.org
```
增加新的源
```
gem sources -a https://gems.ruby-china.com
```
查看源：
```
*** CURRENT SOURCES ***
https://gems.ruby-china.com
```
#####然后再执行：
```
sudo gem install -n /usr/local/bin fastlane --verbose
```
即可正常下载、问题解决！！！

#踩坑二
打包失败还有需要开发者账号配置：
![](https://upload-images.jianshu.io/upload_images/6545546-cecf21dda079d916.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
