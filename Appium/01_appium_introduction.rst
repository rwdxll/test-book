
Appium介绍
=========================

Appium 是一个自动化测试开源工具，支持 iOS 平台和 Android 平台上的原生应用，web 应用和混合应用。

所谓的“移动原生应用”是指那些用 iOS 或者 Android SDK 写的应用。
所谓的“移动 web 应用”是指使用移动浏览器访问的应用（Appium 支持 iOS 上的 Safari 和 Android 上的 Chrome）。
所谓的“混合应用”是指原生代码封装网页视图 —— 原生代码和 web 内容交互。
比如，像 [Phonegap](http://phonegap.com/)，可以帮助开发者使用网页技术写应用，然后用原生代码封装，这些就是混合应用。

重要的是，Appium 是一个跨平台的工具：它允许测试人员使用同样的接口，基于不同的平台（iOS， Android）写自动化测试脚本。
这样大大增加了 iOS 和 Android 测试套件间代码的复用性。

想知道 Appium 如何支持平台，版本和自动化形态的详细信息，请参见[platform support doc](platform-support.cn.md).

Appium 的理念
-------------------------------

为了满足移动自动化需求， Appium 遵循着某种理念。这种理念重点体现于以下 4 个需求：

1. 你无需为了自动化，而重新编译或者修改你的应用。
2. 你不必局限于某种语言或者框架来写和运行测试脚本。
3. 一个移动自动化的框架不应该在接口上重复造轮子。（移动自动化的接口应该统一）
4. 无论是精神上，还是名义上，都必须开源。

Appium 设计
----------------------------------

那么 Appium 架构是如何实现这个哲学呢？为了满足第一条，Appium 真正的工作引擎其实是第三方自动化框架。
这样，我们就不需在你的应用里植入 Appium 特定或者第三方的代码。这就意味着你在测试你将发布的应用。我们
使用以下的第三方框架：

* iOS: 苹果的UIAutomation: https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/
* Android 4.2+: Google's UiAutomator: http://developer.android.com/tools/help/uiautomator/index.html

为了满足第二点，我们把这些第三方框架封装成一套 API，WebDriver: http://docs.seleniumhq.org/projects/webdriver/
WebDriver （也就是 "Selenium WebDriver"） 指定了客户端到服务端的协议。
(参见JSON Wire Protocol: https://w3c.github.io/webdriver/webdriver-spec.html)。
使用这种客户端-服务端的架构，我们可以使用任何语言来编写客户端，向服务端发送恰当的 HTTP 请求。
而且目前已经有大多数流行语言版本的客户端实现了。这也意味着你可以使用任何测试套件或者测试框架。客户端库就是简单的
HTTP 客户，可以以任何你喜欢的方式潜入你的代码。换句话说，Appium 和 WebDriver 客户端不是技术意义上的“测试框架”，
而是“自动化库”。你可以在你的测试环境中随意使用这些自动化库！

事实上 WebDriver 已经成为 web 浏览器自动化的标准，也成了 W3C 的标准 —— https://dvcs.w3.org/hg/webdriver/raw-file/tip/webdriver-spec.html
我们又何必为移动做一个完全不同的呢？
所以我们扩充了WebDriver 的协议:https://github.com/SeleniumHQ/mobile-spec/blob/master/spec-draft.md，
在原有的基础上添加移动自动化相关的 API 方法，这也也满足了第三条理念。

第四条就不用说了，Appium 是开源的，https://github.com/appium/appium

Appium 概念
----------------------------------

**C/S 架构**
Appium 的核心是一个 web 服务器，它提供了一套 REST 的接口。它收到客户端的连接，监听到命令，
接着在移动设备上执行这些命令，然后将执行结果放在 HTTP 响应中返还给客户端。事实上，这种客户端/服务端的
架构打开了许多可能性：比如我们可以使用任何实现了客户端的语言来写我们的测试代码。比如我们可以把服务端放在不同
的机器上。

**Session**
自动化总是在一个 session 的上下文中运行。客户端初始化一个和服务端交互的 session。不同的语言有不同的实现方式，
但是最终都是发送一个附有 “desired capabilities” 的 JSON 对象参数的 POST 请求 “/session” 给服务器。
这时候，服务端就会开始一个自动化的 session，然后返回一个 session ID。
客户端拿到这个 ID，之后就用这个 ID 发送后续的命令。

**Desired Capabilities**
Desired capabilities 是一些键值对的集合 (比如，一个 map 或者 hash）。
客户端讲这些键值对发给服务端，告诉服务端我们想要启动怎样的自动化session。
根据不同的 capabilities 参数，服务端会有不同的行为。比如，我们可以把
`platformName` capability 设置为 `iOS`，告诉 Appium 服务端，我们想要一个
iOS 的 session，而不是一个 Android 的。我们也可以设置 `safariAllowPopups` capability 为 `true`，
确保在 Safari 自动化 session 中，我们可以使用 javascript 来打开新窗口。

**Appium Server**
Appium server 是用 nodejs 写的。我们可以用源码编译或者从 NPM 直接安装。

**Appium 服务端**

Appium 服务端有很多语言库 Java, Ruby, Python, PHP, JavaScript 和 C#，这些库都实现了
Appium 对 WebDriver 协议的扩展。当使用 Appium 的时候，你只需使用这些库代替常规的 WebDriver 库就可以了。

**Appium.app, Appium.exe**

我们提供了 GUI 封装的 Appium server 下载。它封装了运行 Appium server 的所有依赖。
所以你需要担心 Node。而且这个封装包含了一个 Inspector 工具，可以让使用者检查应用的界面层级。
这样写测试用例的时候就非常方便了。

.Appium GUI下载地址: https://bitbucket.org/appium/appium.app/downloads/

.. toctree::
   :maxdepth: 1
   :numbered: 1

   01_Appium_Python_API.rst
   02_Appium_element_locate.rst
   03_Appium_keyevent.rst


本章引用：https://github.com/appium/appium/blob/master/docs/old/cn/intro.cn.md
