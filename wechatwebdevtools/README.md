# 微信开发者工具 探析
```
// package.json
{
    "forceVendor": true,
    "main": "html/index.html",
    "name": "微信web开发者工具",
    "productName": "微信开发者工具",
    "appname": "wechatwebdevtools",
    "version": "1.02.1810190",
    "window": {
        "id": "init",
        "title": "微信开发者工具 v1.02.1810190",
        "icon": "app/images/logo2.png",
        "toolbar": true,
        "width": 334,
        "height": 474,
        "show": false,
        "transparent": false,
        "frame": false
    },
    "inject_js_start": "js/extensions/inject/documentstart/index.js",
    "inject_js_end": "js/extensions/inject/devtools/index.js",
    "chromium-args": "--disable-http2 --disable-devtools --ignore-gpu-blacklist  --enable-experimental-web-platform-features  -ignore-certificate-errors",
    "node-remote": [
        "http://*.open.weixin.qq.com/*",
        "https://*.mp.weixin.qq.com/*",
        "http://127.0.0.1:*/*"
    ],
    "webview": {
        "partitions": [
            {
                "name": "devtools",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "remotedebugdevtools",
                "accessible_resources": [
                    "<all_urls>"
                ],
                "web_accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "gitmanager",
                "accessible_resources": [
                    "<all_urls>"
                ],
                "web_accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "simulator",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "editor",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "appservice",
                "accessible_resources": [
                    "<all_urls>"
                ]
            },
            {
                "name": "htmlwebview",
                "accessible_resources": [
                    "<all_urls>"
                ]
            }
        ]
    },
    "product_string": "wechatwebdevtools"
}
```

基于package.json文件中，我们了解到微信开发者工具的7大功能区：
  1. devtools（开发者工具的调试工具）
  2. remotedebugdevtools（远程真机调试）
  3. gitmanager（基于git的版本管理）
  4. simulator（左侧模拟器，即时显示编码效果）
  5. editor（右侧编辑器，编码区域）
  6. appservice
  7. htmlwebview
  
除此之外，还需要关注两个注入js
  1. "inject_js_start": "js/extensions/inject/documentstart/index.js",
  2. "inject_js_end": "js/extensions/inject/devtools/index.js",
