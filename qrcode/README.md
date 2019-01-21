# 生成qrcode(二维码)的库
1. [qrcode.min.js](https://github.com/davidshimjs/qrcodejs)
2. [jQuery.qrcode.js](https://github.com/jeromeetienne/jquery-qrcode)

## qrcode.min.js
### 基本介绍
1. qrcode.min.js 是生成二维码的库文件，支持跨浏览器（兼容SVG、HTML5 Canvas 和 Table Tag），无外部依赖
2. 使用方式
    ```
    <div id="qrcode"></div>
    <script type="text/javascript" src="path/to/qrcode.min.js"></script>
    <script type="text/javascript">
    // 简单使用
    new QRCode(document.getElementById("qrcode"), "http://jindo.dev.naver.com/collie");

    // 自定义选项
    var qrcode = new QRCode(document.getElementById("qrcode"), {
    	text: "http://jindo.dev.naver.com/collie",
    	width: 128,
    	height: 128,
    	colorDark : "#000000",
    	colorLight : "#ffffff",
    	correctLevel : QRCode.CorrectLevel.H
    });
    </script>
    ```
3. API
    ```
    qrcode.makeCode()
    qrcode.clear()
    qrcode.makeImage()
    ```
### 原理分析
1. 二维码生成算法
    1. 矩阵式二维码，码制为QRCode（Quick-Response code），解码速度快。
    2. [二维码（QR code）基本结构及生成原理](https://blog.csdn.net/u012611878/article/details/53167009)
    3. QR码符号共有40种规格的矩阵，_getTypeNumber根据目标文本长度及纠错级别QRErrorCorrectLevel确定规格。
    4. QRCodeModel.addData，对目标文本进行编码:QR8bitByte
    5. QRCodeModel.make
        - getBestMaskPattern 选择最优的掩码，
        - setupPositionProbePattern 设置位置探测图形，协助扫描软件定位QR码并变换坐标系，让QR码在任意角度被扫描。
        - setupPositionAdjustPattern 设置校正图形，用于进一步校正坐标系，校正标识的数量取决于版本。
        - setupTimingPattern 设置定位图形
        - setupTypeInfo、setupTypeNumber 设置格式和版本信息
        - QRCodeModel.createData 将目标文本生成对应的数据和纠错码字
        - mapData：应用掩码，使得二维码图形中的深色和浅色（黑色和白色）区域能够比率最优的分布，使得二维码图形中的深色和浅色（黑色和白色）区域能够比率最优的分布
![二维码构成](https://github.com/cleverpp/SourceAnalytics/blob/master/qrcode/qrcode-zn.jpg)
2. 源码分析
    1. QRCode 构造函数，主要作用是封装实现，提供接口
    2. QRCodeModel 二维码生成算法实现，最后将text转换成二维数组，该二维数组存的信息是row行col列的位置是否为dark
    3. Drawing 绘制二维码，useSVG为true则使用svg方式绘制，否则判断是否支持canvas，支持则使用canvas方式绘制，不支持才使用table方式绘制。
3. canvas绘制模式下的一些兼容性处理
    1. _getAndroid 通过正则表达式获取当前安卓系统版本号
        ```
        // 有坑，1. 对于非x.y模式的版本无法识别，例如Android 9。 2. 对于未来10及以上版本无法识别。
        sAgent.toString().match(/android ([0-9]\.[0-9])/i)
        ```
    2. Android below 3 doesn't support Data-URI spec， 因此仅android版本大于3才能正常绘制图形
    3. 对android<=2.1通过重写CanvasRenderingContext2D.prototype.drawImage修复bug。

## jquery.qrcode.js
### 基本介绍
1. jquery.qrcode.js 是jquery插件方式，依赖jquery。
2. 使用方式
    ```
    <div id="qrcode"></div>
    <script type="text/javascript" src="path/to/jquery.qrcode.js"></script>
    jquery('#qrcode').qrcode({width: 64,height: 64,text: "size doesn't matter"});
    ```
### 原理分析
1. 二维码生成算法：QRCode与QRCodeModel基本相同。
2. 源码分析
    1. 默认渲染方式canvas，支持table，不支持svg。通过选项opt.render是否为canvas来决定渲染方式
       ```
       var element = options.render == "canvas" ? createCanvas() : createTable();
       ```
    2. QRCode 二维码生成算法实现，最后将text转换成二维数组
3. canvas绘制模式中，对android无特殊兼容性处理

# 其它相关（GenerateQRCode.js）
1. 二维码生成，原理同qrcode.min.js
2. 条形码生成，采用的是code128编码规则
    1. [Code 128 条形码原理和在线生成](https://xoyozo.net/Blog/Details/code128)
    2. [简单解析微信、支付宝，付款码的条形码生成原理](https://www.chenxublog.com/2018/09/22/wechat-alipay-barcode-code128.html)

