---
title: Chrome插件学习
date: 2017-07-13 16:01:11
tags: [Chrome,浏览器插件]
categories: [浏览器,Chrome插件]
comments: true
---
# Chrome插件类型
## 插件
1. browser action：当应用（扩展）的图标是否显示出来是取决于单个的页面时，应当选择page action；
1. page action：当其它情况时可以选择browser action。
1. 应用也可以通过其它方式提供界面，比如加入到上下文菜单，提供一个选项页面或者用一个content script改变页面的显示等。

## WebApp

<!--more-->
# Chrome插件特点
1. 一个应用（扩展）其实是压缩在一起的一组文件，包括HTML，CSS，Javascript脚本，图片文件，还有其它任何需要的文件。 
1. 应用（扩展）本质上来说就是web页面，它们可以使用所有的浏览器提供的API，从XMLHttpRequest到JSON到HTML5全都有。
1. 应用（扩展）可以与Web页面交互，或者通过content script或cross-origin XMLHttpRequests与服务器交互。
1. 应用（扩展）还可以访问浏览器提供的内部功能，例如标签或书签等。

# Chrome插件文件组成
1. 一个manifest文件
1. 一个或多个html文件（除非这个应用是一个皮肤）
1. 可选的一个或多个javascript文件
1. 可选的任何需要的其他文件，例如图片
1. 在开发应用（扩展）时，需要把这些文件都放到同一个目录下。发布应用（扩展）时，这个目录全部打包到一个应用（扩展）名是.crx的压缩文件中。如果使用Chrome Developer Dashboard,上传应用（扩展），可以自动生成.crx文件。

## manifest
```
{
// 必须的字段
  "name": "My Extension",
  "version": "versionString",
  "manifest_version": 2,
  // 建议提供的字段
  "description": "A plain text description",
  "icons": { ... },
  "default_locale": "en",
  // 多选一，或者都不提供
  "browser_action": {...},
  "page_action": {...},
  "theme": {...},
  "app": {...},
  // 根据需要提供
  "background": {...},
  "chrome_url_overrides": {...},
  "content_scripts": [...],
  "content_security_policy": "policyString",
  "file_browser_handlers": [...],
  "homepage_url": "http://path/to/homepage",
  "incognito": "spanning" or "split",
  "intents": {...}
  "key": "publicKey",
  "minimum_chrome_version": "versionString",
  "nacl_modules": [...],
  "offline_enabled": true,
  "omnibox": { "keyword": "aString" },
  "options_page": "aFile.html",
  "permissions": [...],
  "plugins": [...],
  "requirements": {...},
  "update_url": "http://path/to/updateInfo.xml",
  "web_accessible_resources": [...]
}
```