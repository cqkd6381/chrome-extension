---
description: >-
  我正在建立一个Chrome扩展程序，为了让整个事情按照我希望的方式工作，我需要一个外部JavaScript脚本来检测用户是否安装了我的扩展程序。例如：用户安装我的插件，然后用我的脚本进入一个网站。
  该网站检测到我的扩展程序已安装并相应地更新页面。
---

# 检查用户是否安装了Chrome扩展程序

{% hint style="info" %}
####  Chrome现在可以将信息从网站发送到扩展程序。
{% endhint %}

 所以在扩展background.js（content.js不会工作）添加如下内容：

```javascript
chrome.runtime.onMessageExternal.addListener(function(request, sender, sendResponse) {
    if (request) {
        if (request.message) {
            if (request.message == "version") {
                sendResponse({version: 1.0});
            }
        }
    }
    return true;
});
```

 在网站中添加如下脚本：

```javascript
var hasExtension = false;

chrome.runtime.sendMessage(extensionId, { message: "version" }, function (reply) {
    if (reply) {
        if (reply.version) {
            if (reply.version >= requiredVersion) {
                hasExtension = true;
            }
        }
    }else {
      hasExtension = false;
    }
});
```

{% hint style="warning" %}
注意：extensionId为你所要检测的Chrome拓展的ID
{% endhint %}

然后你可以检查hasExtension变量。 唯一的缺点是调用是异步的，所以你必须以某种方式解决这个问题。

编辑：如下所述，您需要向_manifest.json_添加一个条目，列出可以向您的插件发送消息的域。 例如：

```javascript
"externally_connectable": {
    "matches": ["*://localhost/*", "*://your.domain.com/*"]
},
```

 网页通过后台脚本与扩展进行交互。 

manifest.json：

```javascript
"background": {
    "scripts": ["background.js"],
    "persistent": true
},
"externally_connectable": {
    "matches": ["*://(domain.ext)/*"]
},
```

background.js：

```javascript
chrome.runtime.onMessageExternal.addListener(function(msg, sender, sendResponse) {
    if ((msg.action == "id") && (msg.value == id)){
        sendResponse({id : id});
    }
});
```

 page.html：

```markup
<script>
var id = "some_ext_id";
chrome.runtime.sendMessage(id, {action: "id", value : id}, function(response) {
    if(response && (response.id == id)){ //extension installed
        console.log(response);
    }else{ //extension not installed
        console.log("Please consider installig extension");
    }
});
</script>
```

 您可以让该扩展程序[设置一个cookie，](http://code.google.com/chrome/extensions/cookies.html#method-set)并让您的网站JavaScript检查该cookie是否存在并进行相应更新。 当然，这里提到的大多数其他方法可能会受到用户的限制， **除非**您尝试让扩展根据时间戳等创建自定义Cookie，然后让应用程序分析服务器端，以确定它是否是具有扩展名或有人假装通过修改他的cookies来拥有它。

我使用了cookie方法：

在我的manifest.js文件中，我包含了一个只能在我的网站上运行的内容脚本：

```javascript
"content_scripts": [
    {
        "matches": [
            "*://*.mysite.co/*"
        ],
        "js": ["js/mysite.js"],
        "run_at": "document_idle"
    }
], 
```

 在我的js / mysite.js中我有一行：

```javascript
document.cookie = "extension_downloaded=True";
```

并在我的index.html页面中查找该cookie。

```javascript
if (document.cookie.indexOf('extension_downloaded') != -1){
    document.getElementById('install-btn').style.display = 'none';
}
```

我想我会分享我的研究。 我需要能够检测某个文件是否安装了特定的扩展名：///链接才能工作。 我在[blog.kotowicz.net/2012/02/intro-to-chrome-addons-hacking.html](http://blog.kotowicz.net/2012/02/intro-to-chrome-addons-hacking.html)遇到这篇文章这解释了获取扩展名的manifest.json的方法。

我调整了一下代码，并提出：

```javascript
function Ext_Detect_NotInstalled(ExtName,ExtID) {
    console.log(ExtName + ' Not Installed');
    if (divAnnounce.innerHTML  != '')
        divAnnounce.innerHTML = divAnnounce.innerHTML + "<BR>"
        divAnnounce.innerHTML = divAnnounce.innerHTML + 'Page needs ' + ExtName + ' Extension -- to intall the LocalLinks extension click <a href="https://chrome.google.com/webstore/detail/locallinks/' + ExtID +'">here</a>';
}

function Ext_Detect_Installed(ExtName,ExtID) {
    console.log(ExtName + ' Installed');
}

var Ext_Detect = function(ExtName,ExtID) {
    var s = document.createElement('script');
    s.onload = function(){
        Ext_Detect_Installed(ExtName,ExtID);
    };
    s.onerror = function(){
        Ext_Detect_NotInstalled(ExtName,ExtID);
    };
    s.src = 'chrome-extension://' + ExtID + '/manifest.json';
    document.body.appendChild(s);
  }

var is_chrome = navigator.userAgent.toLowerCase().indexOf('chrome') > -1;

if (is_chrome==true){
    window.onload = function() { 
        Ext_Detect('LocalLinks','jllpkdkcdjndhggodimiphkghogcpida');
    };
}
```

 有了这个，你应该可以使用Ext\_Detect（ExtensionName，ExtensionID）来检测任何数量的扩展的安装。

 如果您可以控制Chrome扩展程序，则可以尝试我所做的：

```javascript
// Inside Chrome extension
var div = document.createElement('div');
div.setAttribute('id', 'myapp-extension-installed-div');
document.getElementsByTagName('body')[0].appendChild(div);
```

 接着：

```javascript
// On web page that needs to detect extension
if ($('#myapp-extension-installed-div').length) {

}
```

 这感觉有点冒失，但我无法使用其他方法，我担心Chrome会在这里更改它的API。 这种方法很快就会停止工作，这是值得怀疑的。

