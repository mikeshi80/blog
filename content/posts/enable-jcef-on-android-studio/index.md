+++
title = '如何在Android studio上启用JCEF'
date = 2024-08-01T17:12:13+08:00
categories = '编程经验'
tags = ['Android', 'JCEF']
summary = 'Android studio中默认是无法调用JCEF的功能的，这造成了很多使用了该特性的插件无法使用，本文将介绍如何启用JCEF。'
+++

# 起因

之前为了方便在公司内部环境，使用大语言模型提供的功能，提高开发效率，所以开发了相应的插件，一个是 VSCode 的，一个是 Jetbrains 的 Intellij 系 IDE 的（如 Intellij IDEA，Pycharm 等）。
由于 VSCode 是一个基于前端技术开发的应用，所以在与大语言模型的交互上就直接使用了 Web 前端技术。

而当时在 IDEA，一开始是使用 JEditorPane 来作为显示对话内容的控件，但这个控件只支持简单的 HTML 和 CSS，而不支持 Javascript。这就造成了 Intellij 系插件与 VSCode 的插件在实现上不一样了。

后期为了统一使用前端技术，降低开发成本，所以在 IDEA 中使用 JCEF（Java Chromium Embedded Framework）来作为显示对话内容的控件，这样就可以支持 Javascript 了，两套代码可以共同维护，也方便了 IDEA 和 VSCode 的插件的维护。

其他的一切都很顺利，直到有人想把我这个插件安装在 Android studio 上，竟然发现窗口无法显示其中内容。

# 原因

其实原因在于，出于某种原因，Android studio 中默认是无法调用 JCEF 的功能的。这个因为启动 Android 的 JRE 是一个特殊版本，不支持 JCEF，所以需要手动启用 JCEF。

# 解决方案

可以在启动了 Android studio 之后，通过菜单“Help” -> "Find Action"（也可以通过快捷键 Ctrl+Shift+A），在弹出的搜索框中输入“Choose Boot Java Runtime for the IDE”，然后在弹出的对话框中

{{< insertFigure caption="更新启动JAR" img="boot-jre.png" command="Resize" options="500x" align="center" >}}

点击 New 的下拉选择框，选第一个名字最后带“with JCEF”的选项，然后点击 OK。

这时不要直接关闭或者重启 IDE，后台 IDE 会自动下载刚才选择的 JRE，等下载后系统会要求重启，重启 Android Studio 后，就有 JCEF 的支持了，可以再打开插件窗口确认是否正常运行了。
