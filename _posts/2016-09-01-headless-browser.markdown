---
layout: post
title:  Headless browser
tags: headless-browser
categories: test
---
`Headless browser`是没有界面的浏览器，为其他提供API来访问页面的内容。

下面是来自[WIKI](https://en.wikipedia.org/wiki/Headless_browser)的列表:

```
This is a list of browsers providing a complete or near-complete headless implementation.

PhantomJS – a headless web browser using WebKit layout engine for rendering web pages and JavaScriptCore for executing scripted tests. PhantomJS was originally developed by Ariya Hidayat in 2010 and has gained a wide following and extensive development ecosystem.[10][11][12][13][14]
HtmlUnit – a headless browser written in Java. HtmlUnit uses the Rhino engine to provide JavaScript and AJAX support as well as partial rendering capability.[15][16]
TrifleJS – a headless Internet Explorer scriptable browser using the Trident layout engine for rendering pages and the V8 JavaScript engine for executing scripted tests. TrifleJS uses the same API language as PhantomJS and works by using the .NET WebBrowser object to control whatever version of IE is installed on the machine.[5][17]
Splash – a headless web browser with an HTTP API, Lua scripting support and a built-in IPython (Jupyter)-based IDE. Splash is written in Python and uses WebKit layout engine. Development started at ScrapingHub in 2013; it is partially funded by DARPA.[18][19]
Simulated[edit]
These are browsers that simulate a browser environment. While they are able to support common browser features (HTML parsing, cookies, XHR, some javascript, etc.), they do not render DOM and have limited support for DOM events. They usually perform faster than full browsers, but are unable to correctly interpret many popular websites.[20][21][22]

Zombie.js – a simulated browser environment for Node.js.[23]
ENVJS – a simulated browser environment written in JavaScript for the Rhino engine.[24]
Scriptable[edit]
These are browsers that may still require a user Interface but have programmatic APIs and are intended to be used in ways similar to traditional headless browsers.

SlimerJS – a scriptable browser using Mozilla's Gecko layout engine. SlimerJS uses the same API language as PhantomJS.[25]
```
