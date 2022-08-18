前些天學習了`HTML5`的`<canvas>`元素，今天就來實踐一下，用`canvas`做一個畫板。

首先說一下要實現的功能：
- 切換畫筆顏色
- 調整筆刷粗細
- 清空畫布
- 橡皮擦擦除
- 撤銷操作
- 保存成圖片
- 兼容移動端（支持觸摸）

好了，廢話少說，先看最終效果：

### 準備工作
首先，準備個容器,也就是畫板了。
``` HTML
<canvas id="drawing-board"></canvas>
```
然後初始化js：
```JavaScript
let canvas = document.getElementById("drawing-board");
let ctx = canvas.getContext("2d");
```
我想把畫板做成全屏的，所以接下來設置一下`canvas`的寬高。
```JavaScript
let pageWidth = document.documentElement.clientWidth;
let pageHeight = document.documentElement.clientHeight;

canvas.width = pageWidth;
canvas.height = pageHeight;
```
由於部分IE不支持`canvas`，如果要兼容IE，我們可以創建一個`canvas`，然後使用`excanvas`初始化，針對IE加上[exCanvas.js](http://code.google.com/p/explorercanvas/)，這里我們暫時先不考慮IE。

### 實現一個簡單的畫板
實現思路：監聽鼠標事件，用`drawCircle()`方法把記錄的數據畫出來。

1. 設置初始化當前畫布功能為畫筆狀態，`painting = false`，
2. 當鼠標按下時（`mousedown`），把`painting`設為`true`，表示正在畫，鼠標沒松開。把鼠標點記錄下來。
3. 當按下鼠標的時候，鼠標移動（`mousemove`）就把點記錄下來並畫出來。
4. 如果鼠標移動過快，瀏覽器跟不上繪畫速度，點與點之間會產品間隙，所以我們需要將畫出的點用線連起來（`lineTo()`）。
5. 鼠標松開的時候（`mouseup`），把`painting`設為`false`。

代碼：
```JavaScript
let painting = false;
let lastPoint = {x: undefined, y: undefined};

//鼠標按下事件
canvas.onmousedown = function (e) {
    painting = true;
    let x = e.clientX;
    let y = e.clientY;
    lastPoint = {"x": x, "y": y};
    drawCircle(x, y, 5);
};

//鼠標移動事件
canvas.onmousemove = function (e) {
    if (painting) {
        let x = e.clientX;
        let y = e.clientY;
        let newPoint = {"x": x, "y": y};
        drawLine(lastPoint.x, lastPoint.y, newPoint.x, newPoint.y,clear);
        lastPoint = newPoint;
    }
};

//鼠標松開事件
canvas.onmouseup = function () {
    painting = false;
}

// 畫點函數
function drawCircle(x, y, radius) {
    ctx.save();
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, Math.PI * 2);
    ctx.fill();
}

// 劃線函數
function drawLine(x1, y1, x2, y2) {
    ctx.lineWidth = 3;
    ctx.lineCap = "round";
    ctx.lineJoin = "round";
    ctx.moveTo(x1, y1);
    ctx.lineTo(x2, y2);
    ctx.stroke();
    ctx.closePath();
}
```

### 橡皮擦功能
實現思路
1. 獲取橡皮擦元素
2. 設置橡皮擦初始狀態，`clear = false`。
3. 監聽橡皮擦`click`事件，點擊橡皮擦，改變橡皮擦狀態，`clear = true`。
4. `clear`為`true`時，移動鼠標使用`canvas`剪裁（`clip()`）擦除畫布。

```JavaScript
let eraser = document.getElementById("eraser");
let clear = false;

eraser.onclick = function () {
    clear = true;
};

...
if (clear) {
    ctx.save();
    ctx.globalCompositeOperation = "destination-out";
    ctx.stroke();
    ctx.closePath();
    ctx.clip();
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.restore();
}
...
```
注意，在`canvas`中的裁剪和平時的裁剪功能不一樣在這里，裁剪是指在裁剪區域去顯示我們所畫的圖

### 兼容移動端
實現思路：
1. 判斷設備是否支持觸摸
2. `true`，則使用`touch`事件；`false`，則使用`mouse`事件

代碼：
```JavaScript
...
if (document.body.ontouchstart !== undefined) {
    // 使用touch事件
    anvas.ontouchstart = function (e) {
        // 開始觸摸
    }
    canvas.ontouchmove = function (e) {
        // 開始滑動
    }
    canvas.ontouchend = function () {
        // 滑動結束
    }
}else{
    // 使用mouse事件
    ...
}
...
```
這里需要注意的一點是，在`touch`事件里，是通過`.touches[0].clientX`和`.touches[0].clientY`來獲取坐標的，這點要和`mouse`事件區別開。

### 切換畫筆顏色
實現思路：
1. 獲取顏色元素節點。
2. 點擊顏色返回其顏色值，並賦給畫布的`ctx.strokeStyle`。

代碼：
```JavaScript
let aColorBtn = document.getElementsByClassName("color-item");

for (let i = 0; i < aColorBtn.length; i++) {
    aColorBtn[i].onclick = function () {
    for (let i = 0; i < aColorBtn.length; i++) {
        activeColor = this.style.backgroundColor;
        ctx.strokeStyle = activeColor;
    }
}
```

### 清空畫布
實現思路：
1. 獲取元素節點。
2. 點擊清空按鈕清空canvas畫布。

代碼：
```JavsScript
let reSetCanvas = document.getElementById("clear");

reSetCanvas.onclick = function () {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
};
```

### 調整筆刷粗細
實現思路：
1. 創建input[type=range]
2. 滑動range獲取其value值，並賦給`ctx.lineWidth`

代碼：
```JavaScript
let range = document.getElementById("range");

range.onchange = function(){
    lWidth = this.value;
};
```

### 保存成圖片
實現思路：
1. 獲取canvas.toDateURL
2. 在頁面里創建並插入一個a標簽
3. a標簽href等於canvas.toDateURL，並添加download屬性
4. 點擊保存按鈕，a標簽觸發click事件

代碼：
```JavaScript
let save = document.getElementById("save");

save.onclick = function () {
    let imgUrl = canvas.toDataURL("image/png");
    let saveA = document.createElement("a");
    document.body.appendChild(saveA);
    saveA.href = imgUrl;
    saveA.download = "zspic" + (new Date).getTime();
    saveA.target = "_blank";
    saveA.click();
};
```

### 撤銷
實現思路：
1. 準備一個數組記錄歷史操作
2. 每次使用畫筆前將當前繪圖表面儲存進數組
3. 點擊撤銷時，恢覆到上一步的繪圖表面

代碼：

```
canvas.ontouchstart = function (e) {
     // 在這里儲存繪圖表面
    this.firstDot = ctx.getImageData(0, 0, canvas.width, canvas.height);
    saveData(this.firstDot);
    ...
}

let undo = document.getElementById("undo");
let historyDeta = [];

function saveData (data) {
    (historyDeta.length === 10) && (historyDeta.shift()); // 上限為儲存10步，太多了怕掛掉
    historyDeta.push(data);
}
undo.onclick = function(){
    if(historyDeta.length < 1) return false;
    ctx.putImageData(historyDeta[historyDeta.length - 1], 0, 0);
    historyDeta.pop()
};
```

因為每次儲存都是將一張圖片存入內存，對性能影響較大，所以在這里設置了儲存上限。

### 總結
這里用的知識點主要有：監聽`mouse`、`touch`事件，以及`canvas`的`moveTo()`、`lineTo()`、`stroke()`、`clip()`、`clearRect()`等方法。我相信還有很多效果可以實現，比如說類似噴霧效果，鉛筆字效果，藝術畫效果，等等。日後有時間我會繼續對這個畫板進行優化，增加一些新的功能同時歡迎大家留言提問或者提出批評建議。

