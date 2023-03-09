---
title: BOM和ES6新Api
date: 2021-12-04 14:39:28
tags: 日常学习
cover: /img/Chrome.png
categories: JavaScript
sticky: 4
---

# window

https://wangdoc.com/javascript/bom/window.html

# Navigator

Navigator 对象主要是一些用户设备上的信息，比如操作系统、位置等
下面主要总结一些常用的属性方法

## 属性

下面的属性都是 `navigator.xxx`

- userAgent：设备信息
- plugins：浏览器安装的插件，返回一个数组
- platform：操作系统
- onLine：在线与否，返回布尔值
- geolocation：位置信息
  - `geolocation.getCurrentPosition()`:当前位置，参数可以是一个参数为 pos 的函数，pos 对象有获取时间戳、经纬度等信息
  ```js
  if (window.geolocation) {
    geolocation.getCurrentPosition((pos) => {
      console.log(pos);
    });
  }
  ```
  - `geolocation.watchPosition()`：监听用户位置变化，相当于 GPS 的效果，与之对应的还有 clearWatch 清除监听
- `clipboard`，剪切板对象，可以访问用户的剪切板进行读写操作。剪切板操作是异步的，方法返回一个 promise 对象，常用的方法有：
  - `Clipboard.readText()`
  - `Clipboard.read()`
  - `Clipboard.writeText()`
  - `Clipboard.write()`
    这些 api 的具体操作可以参考<a href="https://wangdoc.com/webapi/clipboard.html">网道-剪切板 API</a>

# Screen

Screen 主要是一些屏幕信息。注意是屏幕而不是网页大小，也就是说像 height 这些属性，当网页大小变化时是不会改变的

## 属性

- height/width：是个定值，表示屏幕高、宽，单位是 px
- availHeight/availWidth：浏览器窗口宽高

# Cookie

## 特点

关于 cookie 的详细及原理可以参考网络的那篇博客；
一般 cookie 都会含有这几项信息：

- Cookie 的名字
- Cookie 的值（真正的数据写在这里面）
- 到期时间（超过这个时间会失效）
- 所属域名（默认为当前域名）
- 生效的路径（默认为当前网址）

这里说一下 cookie 的生效条件：

- 端口和协议不会影响读取
- 域名不同是不能读取的，比如 example.com 不能读取 test.example.com 的 cookie
- 可以通过规定 cookie 的 domain 属性设置为较高一级的域名让所有子级域名访问（比如设置为 domain:example.com 就可以让 test.example.com 访问到）

## 属性

### HTTP 头中的 cookie

```
Set-Cookie: <cookie-name>=<cookie-value>; domain=<domain-value>; Secure; HttpOnly; path=/<path-name>; Expires=<Expires-Time>
```

比如：

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```

- 每个属性间用分分隔，一般等号两边不加空格
- 只要有一个属性不同，就会生成一个全新的 Cookie，而不是替换掉原来那个 Cookie。

具体属性分析：

- Expires 属性指定一个具体的到期时间，是 UTC 格式，可以使用`Date.prototype.toUTCString()`进行格式转换。
- Max-Age 属性指定从现在开始 Cookie 存在的秒数，是数字
- Domain 属性指定 Cookie 属于哪个域名
- Path 是路径，就是你在这个网站下的路径信息。通常设置为`path=/`，意思就是在该网站下的所有路径都能访问到
- Secure 属性指定 HTTPS 下才会发送至服务器
- HttpOnly 属性指定该 Cookie 无法通过 JavaScript 脚本拿到

### js 中的 cookie

document.cookie 属性用于读写当前网页的 Cookie。

- 返回一个字符串，比如`"foo=bar; baz=bar"`，可以用`split(';')`分隔来读取
- `document.cookie = "key=value"`可以写入，写入属性是**添加**而并不是覆盖
- 写入的时候，Cookie 的值必须写成 `key=value` 的形式。注意，**等号两边不能有空格**，必须对分号、逗号和空格进行转义

读取示例：

```js
let cookies = document.cookie.split("; "); //注意后面留一个空格，因为每个分号分割的cookie之间其实多了一个空格
```

写入：写入要考虑转义，使用 URL 对象的`encodeURIComponent`方法可以转义。转义的规则和 url 一样

```js
let userObj = {
  name: "name",
  id: "001",
};

let newCookies = Object.entries(userObj)
  .map((entry) => entry.map((value) => encodeURIComponent(value)).join("="))
  .join("; ");
```

另外，写入 cookie 时也可以同时写入这个 cookie 的属性，即 expires、security 等属性。比如：

```js
document.cookie =
  "fontSize=14; " +
  "expires=" +
  someDate.toGMTString() +
  "; " +
  "path=/subdirectory; " +
  "domain=*.example.com";
```

属性只可以写入而不能读取。属性的读取只能依靠 http 请求中在服务端获取

# history

History 接口允许操作浏览器的曾经在标签页或者框架里访问的会话历史记录。
## 属性

- length：访问的数量
- state：表示history的状态，通过pushState等方式修改。这个state其实和history对象本身没有任何关系，完全由开发者通过pushState方法创建，传入什么对象，state就会是什么对象。

## 方法

- `History.back()`
- `History.forward()`
- `History.go()`
- history.pushState
- history.replaceState

其中，后两个是最常用的方法，用于修改浏览器的url显示以及浏览记录。
假设在 `http://mozilla.org/foo.html` 页面的 console 中执行了以下 JavaScript 代码：

```js
window.onpopstate = function(e) {
   alert(2);
}

let stateObj = {
    foo: "bar",
};
history.pushState(stateObj, "page 2", "bar.html");
```

这将使浏览器地址栏显示为 `http://mozilla.org/bar.html`，但并不会导致浏览器加载 bar.html ，甚至不会检查bar.html 是否存在。
同时history.state对象将会变成`{foo: "bar"}`
如果这时点击后退按钮，就会回到`http://mozilla.org/foo.html`。也就是相当于为浏览器添加了一条历史记录
replaceState和pushState最大的差别在于pushState会像push栈一样添加历史记录，而replace则是替换。

### onpopstate事件

调用 `history.pushState()` 或者 `history.replaceState()` 不会触发 popstate 事件。但是仍然可以在popstate事件中获取到最新的由这两个api设置的state对象。
popstate 事件只会在浏览器某些行为下触发，比如点击后退按钮，或者在 JavaScript 中调用 history.back/forward 之类的方法

```js
window.onpopstate = function(event) {
  alert("location: " + document.location + ", state: " + JSON.stringify(event.state));
};

history.pushState({page: 1}, "title 1", "?page=1");
history.pushState({page: 2}, "title 2", "?page=2");
history.replaceState({page: 3}, "title 3", "?page=3");
history.back(); // 弹出 "location: http://example.com/example.html?page=1, state: {"page":1}"
history.back(); // 弹出 "location: http://example.com/example.html, state: null
history.go(2);  // 弹出 "location: http://example.com/example.html?page=3, state: {"page":3}
```
# Location

## 属性

```js
// 当前网址为
// http://user:passwd@www.example.com:4097/path/a.html?x=111#part1
document.location.href;
// "http://user:passwd@www.example.com:4097/path/a.html?x=111#part1"
document.location.protocol;
// "http:"
document.location.host;
// "www.example.com:4097"
document.location.hostname;
// "www.example.com"
document.location.port;
// "4097"
document.location.pathname;
// "/path/a.html"
document.location.search;
// "?x=111"
document.location.hash;
// "#part1"
document.location.username;
// "user"
document.location.password;
// "passwd"
document.location.origin;
// "http://user:passwd@www.example.com:4097"
```

location.href 直接写入会跳转到新网址，可以用这个来滚动到新锚点，比如`document.location.href = '#top';`

## 方法

- Location.assign()，使得浏览器立刻跳转到新的 URL，相当于 push
- Location.replace()，跳转，参数会被添加到当前路径后面
- Location.reload()，强制刷新

# URL

## 转义

URL 的字符只能是下面的几种：

- URL 元字符
  - 分号（;）
  - 逗号（,）
  - 斜杠（/）
  - 问号（?）
  - 冒号（:）
  - at（@）
  - &- 等号（=）
  - 加号（+）
  - 美元符号（$）
  - 井号（#）
- 语义字符
  - a-z
  - A-Z
  - 0-9
  - 连词号（-）
  - 下划线（\_）
  - 点（.）
  - 感叹号（!）
  - 波浪线（~）
  - 星号（\*）
  - 单引号（'）
  - 圆括号（()）

不是上面的都会转义，根据操作系统的默认编码，将每个字节转为百分号（%）加上两个大写的十六进制字母
比如：

> UTF-8 的操作系统上，`http://www.example.com/q=春节` 这个 URL 之中，汉字“春节”不是 URL 的合法字符，所以被浏览器自动转成http://www.example.com/q=%E6%98%A5%E8%8A%82。其中，“春”转成了%E6%98%A5，“节”转成了%E8%8A%82。这是因为“春”和“节”的 UTF-8 编码分别是 E6 98 A5 和 E8 8A 82，将每个字节前面加上百分号，就构成了 URL 编码。

转义可以用 js 的这几种方法：

- `encodeURI()`：用于转码整个 URL，比如：

```js
encodeURI("http://www.example.com/q=春节");
// "http://www.example.com/q=%E6%98%A5%E8%8A%82"
```

- `encodeURIComponent()`：会转码除了语义字符之外的**所有字符**，即元字符（比如`/`）也会被转码
- `decodeURI()`：encodeURI 的逆运算
- `decodeURIComponent()`：同理，encodeURIComponent 的逆运算

## URL 实例方法

`const url = new URL("http://www.example.com")`可以创建 URL 的实例
URL 的属性和 location 差不多，不再赘述，下面是一些方法：

- `URL.createObjectURL()`：用来为上传/下载的文件、流媒体文件生成一个 URL 字符串。这个字符串代表了 File 对象或 Blob 对象的 URL。
  比如通过 input type="file"拿到了图片的 file 对象，可以上传的文件生成一个 URL 字符串，作为 img 元素的图片来源。

```js
function handleFiles(files) {
  for (var i = 0; i < files.length; i++) {
    var img = document.createElement("img");
    img.src = window.URL.createObjectURL(files[i]);
    div.appendChild(img);
  }
}
```

生成的 URL 像这样：

```
blob:http://localhost/c745ef73-ece9-46da-8f66-ebes574789b1
```

还可以配合 a 标签的 download 属性使得链接成为一个下载。当点击这个标签时，就会下载一个文件。

```js
input.addEventListener("change", (e) => {
  const file = e.target.files[0];
  const url = URL.createObjectURL(file);
  link.download = file.name;
  link.href = url;
});
```

# ArrayBuffer

二进制数组很像 C 语言的数组，允许开发者以数组下标的形式，直接操作内存，大大增强了 JavaScript 处理二进制数据的能力，使得开发者有可能通过 JavaScript 与操作系统的原生接口进行二进制通信。

二进制数组由三类对象组成：

- `ArrayBuffer`：代表内存之中的一段二进制数据，是最基本的二进制对象，表示对固定长度的连续内存空间的引用。

如果通过`new ArrayBuffer(n)`的形式创建一个长度为 n 的 buffer，就会分配一个连续的、长度为 n 的连续内存空间。
同一段内存，不同数据有不同的解读方式，这就叫做“视图”（view）；可以通过“视图”进行操作。“视图”部署了数组接口，这意味着，可以用数组的方法操作内存。

---

- `TypedArray`视图：共包括 9 种类型的视图，比如 Uint8Array（无符号 8 位整数）数组视图, Int16Array（16 位整数）数组视图, Float32Array（32 位浮点数）数组视图等等。
  视图对象本身并不存储任何东西。它是一副“眼镜”，透过它来解释存储在 ArrayBuffer 中的字节。我们选择用什么值来表示这些字节，就称作选择什么视图。通常情况下都是用二进制数字来表示：

  - Uint8Array：将 ArrayBuffer 中的每个字节视为 0 到 255 之间的单个数字（每个字节是 8 位，即表示 0~2^8-1）。这称为 “8 位无符号整数”。
  - Uint16Array：将每 2 个字节视为一个 0 到 65535 之间的整数。这称为 “16 位无符号整数”。
  - Uint32Array：将每 4 个字节视为一个 0 到 4294967295 之间的整数。这称为 “32 位无符号整数”。
  - Float64Array：将每 8 个字节视为一个 5.0x10-324 到 1.8x10308 之间的浮点数。
    ![](https://pic1.imgdb.cn/item/634fca3b16f2c2beb16d2d7f.jpg)

  普通数组与 TypedArray 数组的差异主要在以下方面。

  - TypedArray 数组的所有成员，都是同一种类型。
  - TypedArray 数组的成员是连续的，不会有空位。
  - TypedArray 数组成员的默认值为 0。比如，new Array(10)返回一个普通数组，里面没有任何成员，只是 10 个空位；new Uint8Array(10)返回一个 TypedArray 数组，里面 10 个成员都是 0。
  - TypedArray 数组只是一层视图，本身不储存数据，它的数据都储存在底层的 ArrayBuffer 对象之中，要获取底层对象必须使用 buffer 属性。
  - TypedArray 具有常规的 Array 方法，可以遍历（iterate），map，slice，find 和 reduce 等。

---

- `DataView`视图：可以自定义复合格式的视图，比如第一个字节是 Uint8（无符号 8 位整数）、第二、三个字节是 Int16（16 位整数）、第四个字节开始是 Float32（32 位浮点数）等等，此外还可以自定义字节序。

很多浏览器操作的 API，用到了二进制数组操作二进制数据，比如 xhr 的响应和请求、Canvas 的二进制像素数据、ws 的二进制数据，以及 Blob 和 File 对象。

## 应用

1. 可以设置 xhr 的返回值是二进制数据。把返回类型（responseType）设为 arraybuffer；如果不知道具体的二进制数据类型，就设为 blob。

```js
let xhr = new XMLHttpRequest();
xhr.open("GET", someUrl);
xhr.responseType = "arraybuffer";

xhr.onload = function () {
  let arrayBuffer = xhr.response;
  // ···
};

xhr.send();
```

2. websocket 和 fetch 都可以收发二进制数据。其中 fetch 的返回数据本身就是一个 ArrayBuffer 对象，因此需要调用`.json()`方法转为文本对象。

3. 如果知道文件对象的类型，就可以把 Blob 或者 File 对象转为 ArrayBuffer。通过`FileReader.readAsArrayBuffer`即可。

# Blob

Blob 对象表示一个二进制文件的数据内容，比如一个图片文件的内容就可以通过 Blob 对象读写。它通常用来读写文件，它的名字是 Binary Large Object （二进制大型对象）的缩写。也就是说它可以用于**操作二进制文件**
Blob 由一个可选的字符串 type（通常是 MIME 类型）和 blobParts 组成。blobParts 可以是一个 blob 对象数组或字符串数组，也可以是混合的
![](https://pic.imgdb.cn/item/622d738c5baa1a80ab200a7c.jpg)

可以用 `new Blob()`生成 Blob 对象，通常有两个参数：

- 第一个参数是二进制对象**数组**或字符串**数组**，确切说是`Blob/BufferSource/String` 类型的值的数组。
- 第二个参数是一个只有一个属性 type 的的对象，用于规定生成格式（MIME 格式），比如`{type:'text/html'}`。如果通过 fetch 等方法 post 一个 blob，type 会自动变成`Content-type`的类型，不用单独设置。

```js
const htmlFragment = ['<a id="a"><b id="b">hey!</b></a>'];
const myBlob = new Blob(htmlFragment, { type: "text/html" });
```

## 属性和方法

- size/type: 获取对象大小和格式
- `slice()`：提取 Blob 片段；类似于 `array.slice`，参数也允许是负数
- `stream()`：把 blob 当作一个流来读取，返回一个`readableStream`对象，可以使用`getReader`方法在其上创建可读流。

## Blob 用作 URL

Blob 可以很容易用作 `<a>`、`<img>` 或其他标签的 URL。
核心方法是`URL.createObjectURL`，参数是一个 Blob 对象，返回一个唯一的 URL，形式为 `blob:<origin>/<uuid>`。
这个 url 可以被用在标签上；浏览器为这个 url 形成了映射，使得这个 url 可以直接访问该 blob 对象，进行读取解析、下载等操作。浏览器处理 Blob URL 就跟普通的 URL 一样，如果 Blob 对象不存在，返回 404 状态码；如果跨域请求，返回 403 状态码。Blob URL 只对 GET 请求有效，如果请求成功，返回 200 状态码。由于 Blob URL 就是普通 URL，因此可以下载。
这是一个下载的例子：

```html
<a download="hello.txt" href="#" id="link">Download</a>

<script>
  let blob = new Blob(["Hello, world!"], { type: "text/plain" });

  link.href = URL.createObjectURL(blob);
</script>
```

上面的代码点击`<a>`后将会下载一个 hello.txt 文件，其中内容为`"Hello, world!"`。

## Blob 转换为 base64

上面的另外一种解决方式是，可以将 Blob 转化为 Base64 格式，然后将 Base64 格式的 blob 对象应用在需要的地方。

核心方法是`FileReader.readAsDataURL()`，参数传入 Blob 对象，异步返回一个 Base64 data-url。

> base64 实际上也是一种 url

```js
let blob = new Blob(["Hello, world!"], { type: "text/plain" });

let reader = new FileReader();
reader.readAsDataURL(blob); // 将 Blob 转换为 base64 并调用 onload

reader.onload = () => {
  link.href = reader.result; // data url
};
```

## 操作文件

- 文件选择器返回一个 FileList 对象，该对象是一个类似数组的成员，每个成员都是一个 File 实例对象。**File 实例对象是一个特殊的 Blob 实例**，增加了 `name` 和 `lastModifiedDate` 属性。
- AJAX 请求时，如果指定**responseType 属性为 blob**，下载下来的就是一个 Blob 对象。

取得 **Blob 对象**以后，可以通过 FileReader 对象，读取 Blob 对象的内容。也就是说，文件选择器选择的文件、网络请求获取的 blob 都可以通过下面的方法读取

- `FileReader.readAsText()`：返回文本，需要指定文本编码，默认为 UTF-8。
- `FileReader.readAsArrayBuffer()`：返回 ArrayBuffer 对象。
- `FileReader.readAsDataURL()`：返回 Data URL。这个比较常用于图片 base64 格式的转换，
- `FileReader.readAsBinaryString()`：返回原始的二进制字符串。

# File

由上面所说，File 实例对象是一个特殊的 Blob 实例，所以文件的相关属性和 blob 对象类似。
file 对象广泛存在于网页中和文件上传、下载等相关 api 中，通过`<input type="file">`获取的文件对象就是 file 或 file 数组类型。

## 获取

- 可以通过文件选择器获取，通过文件选择器的实例.files 获取 file 数组，每个元素都是 file 对象

```js
const file = document.getElementsByTagName("input").files[0];
```

- 通过 `File()`构造函数，`File()`构造函数接受三个参数。
  - array：一个数组，成员可以是二进制对象或字符串，表示文件的内容。
  - name：字符串，表示文件名或文件路径。
  - options：配置对象，设置实例的属性。该参数可选。

其中第三个参数配置对象`options`，可以设置两个属性。

- type：字符串，表示实例对象的 MIME 类型，默认值为空字符串。
- lastModified：时间戳，表示上次修改的时间，默认为 Date.now()。

举个栗子：

```js
const file = new File(["foo"], "foo.txt", {
  type: "text/plain",
});
```

## 属性

- `File.lastModified`：最后修改时间
- `File.name`：文件名或文件路径
- `File.size`：文件大小（单位字节）
- `File.type`：文件的 MIME 类型，就是 type 属性的值，比如 text/plain

## FileReader

FileReader 对象用于读取 File 对象或 Blob 对象所包含的文件内容，**比较常用**
通过 new 的方式创建一个 fileReader 实例：

```js
const fr = new FileReader();
```

### 属性

以下属性都是 FileReader 的实例上的，方法同理；

- `FileReader.error`：读取文件时产生的错误对象
- `FileReader.readyState`：整数，表示读取文件时的当前状态。一共有三种可能的状态，0 表示尚未加载任何数据，1 表示数据正在加载，2 表示加载完成。
- `FileReader.result`：读取完成后的文件内容，有可能是字符串，也可能是一个二进制实例。注意这个是拿到**读取结果**的属性
- `FileReader.onerror`：error 事件（读取错误）的监听函数。
- `FileReader.onload`：load 事件（读取操作完成）的监听函数，**通常在这个函数里面使用 result 属性，拿到文件内容**。
- `FileReader.onprogress`：progress 事件（读取操作进行中）的监听函数。（可以用这个做个进度条

### 方法

- `FileReader.abort()`：终止
- `FileReader.readAsArrayBuffer()`：以 ArrayBuffer 的格式读取文件，返回一个 ArrayBuffer 实例。
- `FileReader.readAsBinaryString()`：返回原始的二进制字符串。
- `FileReader.readAsDataURL()`：参数是 file 对象，返回一个 Data URL 格式（Base64 编码）的字符串，文件转 base64 的方法；
- `FileReader.readAsText()`：读取完成后，result 属性将返回文件内容的**文本**字符串。该方法的第一个参数是代表文件的 file 实例，第二个参数是可选的，表示文本编码，默认为 UTF-8。

FileReader 的读取方法默认都是异步的，但是不需要手动异步操作，只需要先执行 Read 相关方法，然后通过`onload`、`onerror`回调的形式，从 event 中获取数据
举个栗子，读取图片并转成 base64：

```js
const file = document.getElementById("files").files[0];
const fr = new FileReader();
function readPicAsBase(file) {
  fr.readAsDataURL(file);
  fr.onload = (e) => {
    console.log(e.target.result);
  };
}
```

注意这个例子中的 e 类型是 `FileReaderEvent`，target 类型可以是 `FileReaderEventTarget`；或者直接用 Event

# FormData

`FormData()`构造函数的参数是一个**DOM 的表单元素**，构造函数会自动处理表单的键值对。这个参数是可选的，如果省略该参数，就表示一个空的表单。

## 属性

通过构建得到 formdata 的实例：

```js
let myForm = document.getElementById("myForm");
let formData = new FormData(myForm);
// 获取某个控件的值
formData.get("username"); // ""
// 设置某个控件的值
formData.set("username", "张三");
formData.get("username"); // "张三"
```

> 注意：**直接打印 formData 实例是看不到的，想要访问必须通过以下方法：**

## 方法

- `FormData.get(key)`：获取指定键名对应的键值，参数为键名。如果有多个同名的键值对，则返回第一个键值对的键值。
- `FormData.getAll(key)`：返回一个数组，表示指定键名对应的所有键值。如果有多个同名的键值对，数组会包含所有的键值。
- `FormData.set(key, value)`：设置指定键名的键值，参数为键名。如果键名不存在，会添加这个键值对，否则会更新指定键名的键值。如果第二个参数是文件，还可以使用第三个参数，表示文件名。
- `FormData.delete(key)`：删除一个键值对，参数为键名。
- `FormData.append(key, value)`：添加一个键值对。如果键名重复，则会生成两个相同键名的键值对。如果第二个参数是文件，还可以使用第三个参数，表示文件名。
- `FormData.has(key)`：返回一个布尔值，表示是否具有该键名的键值对。
- `FormData.keys()`：返回一个遍历器对象，用于 for...of 循环遍历所有的键名。比如`for(let key of formData.keys())`
- `FormData.values()`：返回一个遍历器对象，用于 for...of 循环遍历所有的键值。
- `FormData.entries()`：返回一个遍历器对象，用于 for...of 循环遍历所有的键值对。如果直接用 for...of 循环遍历 FormData 实例，默认就会调用这个方法。

其中 append 方法一般用于手动创建表单对象添加值

> 注意：直接 get(key)获取的是**空值（""）**，一般需要 set 或者 append 设置了值才能拿到

有了 formdata，就可以通过生成 formData 的形式整个获取表单数据，在表单内的所有含有 name 的 input 元素的输入都会被捕获

```html
<body>
  <form>
    <label for="user-name">name</label>
    <input id="user-name" name="username" />

    <label for="user-password">password</label>
    <input id="user-password" name="password" />

    <input type="checkbox" id="check" name="check" />
    <input type="submit" value="submit" />
  </form>
</body>
<script>
  const form = document.forms[0];

  form.addEventListener("submit", (e) => {
    e.preventDefault();
    const formData = new FormData(form);
    const data = Object.fromEntries(formData.entries());

    //checkbox默认值为on，不勾选则没有默认值，因此需要手动加上checkbox的值
    if (!data.check) data.check = false;
    else data.check = true;

    console.log(data);
  });
</script>
```

## 表单校验

input 上面可以有一些限制来校验表单，常用的如下：

- pattern：正则
- max/min：针对数值最大最小
- maxlength/minlength：针对字符串长度最大最小
- required：必填项

通过给 input 加上:valid 伪元素可以使校验不通过的元素产生对应样式
校验不通过时提交不会触发

## 上传文件

如果上传需要 multipart/form-data 格式，就需要构造 formdata 实例并添加要上传的文件
比如：

```js
<input type="file" id="file" />;

const file = document.getElementById("file")[0];
const fd = new FormData();
if (file.type.match("image.*")) {
  formData.append("photos", file, file.name); //参数是键名、值和文件名
}
axios({
  type: "POST",
  headers: {
    contentType: file.type,
  },
  data: file,
  url: "/api",
});
```

# Storage

Storage 接口用于脚本在浏览器保存数据。两个对象部署了这个接口：`window.sessionStorage`和`window.localStorage`。

- `sessionStorage`保存的数据用于浏览器的一次会话（session），当会话结束（通常是窗口关闭），数据被清空；
- `localStorage`保存的数据长期存在，下一次访问该网站的时候，网页可以直接读取以前保存的数据。除了保存期限的长短不同，这两个对象的其他方面都一致。

保存的数据都以“键值对”的形式存在。也就是说，每一项数据都有一个键名和对应的值。所有的数据都是以**文本**格式保存。很像 Cookie 的强化版，能够使用大得多的存储空间。

## Storage 的属性和方法

Storage 接口只有一个属性。
`Storage.length`：返回保存的数据项个数。

```js
localStorage.setItem("foo", "a");
localStorage.setItem("bar", "b");
localStorage.setItem("baz", "c");

localStorage.length; // 3
```

方法主要有 5 个，即对存储的获取、删除、移除、全部清除、获取键值

### `setItem()`

添加一个值，但是直接按照类似对象的写法添加也可以。

> 点访问和括号访问方法可以直接修改已存储的对象

```js
localStorage.foo = "123";
localStorage["foo"] = "123";
localStorage.setItem("foo", "123");
```

存入的值和键默认都是字符串，如果不是则会调用`toString()`转成字符串，因此存入一个非字符串实际上的值是`[xxx Object]`。
通常存储的数据通过`JSON.stringfy`转为 json 格式字符串

```js
sessionStorage.setItem(3, { foo: 1 });
sessionStorage.getItem("3"); // "[object Object]"
```

### `Storage.getItem()`

`Storage.getItem()`方法用于读取数据。它只有一个参数，就是键名。如果键名不存在，该方法返回 null。

```js
sessionStorage.getItem("key");
localStorage.getItem("key");
```

键名应该是一个字符串，否则会被自动转为字符串。

### `Storage.removeItem()`

`Storage.removeItem()`方法用于清除某个键名对应的键值。它接受键名作为参数，如果键名不存在，该方法不会做任何事情。

```js
sessionStorage.removeItem("key");
localStorage.removeItem("key");
```

### `Storage.clear()`

`Storage.clear()`方法用于清除所有保存的数据。该方法的返回值是 undefined。

```js
sessionStorage.clear();
localStorage.clear();
```

### `Storage.key()`

`Storage.key()`方法接受一个整数作为参数（从零开始），返回该位置对应的键名。

```js
sessionStorage.setItem("key", "value");
sessionStorage.key(0); // "key"
```

### 事件

Storage 接口储存的数据发生变化时，会触发 storage 事件，可以指定这个事件的监听函数。

```js
window.addEventListener("storage", (se) => {
  console.log(se.key); //发生变化的键
  console.log(se.newValue);
  console.log(se.oldValue);
  console.log(se.storageArea);
  console.log(se.url);
});
```

# IndexedDB

IndexedDB 就是浏览器提供的本地数据库，它可以被网页脚本创建和操作。IndexedDB 允许储存大量数据，提供查找接口，还能建立索引。这些都是 LocalStorage 所不具备的。就数据库类型而言，IndexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。
IndexedDB 并不能替代服务端使用的数据库，更多是用来作为 Storage 存储空间不足的替代和扩充，可以存储二进制文件，并且是异步的，从性能上要比 storage 强大很多。

## 1. 打开数据库

使用 `indexedDB.open()`打开数据库

```js
const request = window.indexedDB.open(databaseName);
```

- 如果指定的数据库不存在，就会新建
- 参数是**字符串**，表示数据库名字
- 返回 `IDBRequest` 对象，通过三种事件 `onerror`、`onsuccess`、`onupgradeneeded`，处理打开数据库的操作结果；`onsuccess` 事件可以通过`request.result` 属性拿到数据库对象

```js
const request = window.indexedDB.open(databaseName);
let db;
request.onsuccess = () => {
  db = request.result;
};
```

## 2. 新建数据库

新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同之处在于，后续的操作主要在`upgradeneeded`事件的监听函数里面完成，因为这时版本从无到有，所以会触发这个事件。
新建数据库以后，第一件事是新建对象仓库（即新建表）

```js
request.onupgradeneeded = (e) => {
  db = e.target.result;
  if (!db.objectStoreNames.contains("person")) {
    objectStore = db.createObjectStore("person", { keyPath: "id" });
  }
};
```

- createObjectStore 参数：person 表示表名；后面的对象表示主键名"id"；
- 如果没有合适的主键，可以使用自动生成，`{ autoIncrement: true }`

## 3. 新建索引

索引相当于表格的表头；不同于其他数据库，indexedDB 并不需要指明类型。

```js
objectStore.createIndex("name", "name", { unique: false });
objectStore.createIndex("age", "age", { unique: true });
```

参数分别为：索引名称、索引所在的属性、配置对象（说明该属性是否包含重复的值）

## 4. 新增数据

```js
function add() {
  const request = db
    .transaction(["person"], "readwrite")
    .objectStore("person")
    .add({ id: 1, name: "张三", age: 24 });

  request.onsuccess = function (event) {
    console.log("数据写入成功");
  };

  request.onerror = function (event) {
    console.log("数据写入失败");
  };
}
```

- 首先通过 transaction 创建一个事务，参数 1 是指定的数据表；参数 2 是方式"readwrite"表示读写，不写表示读
- request 的 objectStore 方法可以拿到数据表对象，注意传参表名
- add 方法添加一个对象，注意如果定义了主键就要有主键，并且后面的索引要和建立的索引对应

## 5. 读取数据

```js
function read() {
   const req = db.transaction(['person'])
    .objectStore('person');
    .get(1);
    .onerror = function(event) {
     console.log('事务失败');
    }
   .onsuccess = function(event) {
      if (req.result) {
        console.log('Name: ' + req.result.name);
        console.log('Age: ' + req.result.age);
      } else {
        console.log('未获得数据记录');
      }
    }
}
```

- 同样是先建立会话，然后用 objectStore 获取表；
- 使用 get 方法，参数是主键
- get 之后可以通过 req.result 可以拿到结果，并可以用 onsuccess 和 onerror 来监听

## 6. 更新数据

使用 put 方法更新

```js
function add() {
  const request = db
    .transaction(["person"], "readwrite")
    .objectStore("person")
    .put({ id: 1, name: "李四", age: 18 });

  request.onsuccess = function (event) {
    console.log("数据更新成功");
  };

  request.onerror = function (event) {
    console.log("数据更新失败");
  };
}
```

## 7. 删除数据

```js
function remove() {
  var request = db
    .transaction(["person"], "readwrite")
    .objectStore("person")
    .delete(1);

  request.onsuccess = function (event) {
    console.log("数据删除成功");
  };
}
```

# 其他小知识点

## for await of

异步迭代器，用于迭代一个包含异步任务的数组。

```js
function fn(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(`${time}毫秒后我成功啦！！！`);
    }, time);
  });
}
async function asyncFn() {
  const arr = [fn(3000), fn(1000), fn(1000), fn(2000), fn(500)];
  for await (let x of arr) {
    //输出的是一个promise
    console.log(x);
  }
}
```

相当于对数组中的每一项依次执行 await；但是前提是数组的项必须要是 promise，就和 promise.all 类似。
这个api的主要作用是针对iterator对象。由于其使用了遍历for of，因此可以直接处理iterator或generator对象。

# Generator

Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。

## 基本 Generator

形式上，Generator 函数是一个普通函数，但是有两个特征。

- function 关键字与函数名之间有一个星号；
- 函数体内部使用 yield 表达式，定义不同的内部状态

```js
function* generate() {
  console.log("hello world");
  yield "hello";
  yield "world";
  yield 3 + 3;
  return "stop";
}

const generateRes = generate();
```

generator 函数执行会返回一个可迭代对象。如果不用迭代器或者用 next 等方法，则函数内部的代码不会执行。

因为是可迭代对象，所以可以直接遍历函数返回值，或者用展开运算符展开到数组。

```js
function* generate() {
  console.log("hello world");
  yield "hello";
  yield "world";
  yield 3 + 3;
  return "stop";
}
const generateRes = generate();

generateRes.next(); // {value:'hello',done:false}
generateRes.next(); // {value:'world',done:false}
generateRes.next(); // {value:6,done:false}
generateRes.next(); // {value:'stop',done:true}

for (let val of generateRes) {
  console.log(val);
}

[...generateRes];
```

### next 方法

next 方法就像迭代对象的 next 方法一样，会向后迭代一步。在 generator 函数中它会从函数头部或上一次停下来的地方开始执行，直到遇到下一个 yield 表达式（或 return 语句）为止。

next 函数每次执行，都会按顺序停在第 n 个 yield 语句。比如第一次执行 next 就会停在从头开始的第一个 yield 语句上；最后一次执行会停在 return，或者走过最后一个 yield 语句（即函数结束）。

在 generator 函数中，next 方法会返回一个对象，包括两个属性：

- value：即当前执行的语句的 yield 的后面部分的返回值
- done：如果执行到 return 或者没有返回值时走过了最后一个 yield，则 done 为 true。其他情况为 false

next 方法可以传入一个参数，作为**上一个 yield 语句的结果**。比如：

```js
function* foo(x) {
  var y = 2 * (yield x + 1);
  var z = yield y / 3;
  return x + y + z;
}

const b = foo(5);
b.next(); // { value:6, done:false }
b.next(12); // { value:8, done:false }
b.next(13); // { value:42, done:true }
```

这里第一次的 value 本来为 6（即 x+1），第二次调用时传入 12，作为上一句 yield 的返回值（即 y = 2 * 12），因此第二个 yield 返回 8（2*12/3）。第三次设置 z 的值为 13，得到返回值为 42（24+5+13）

因为它的参数是设置上一句的结果，因此第一个 next 的调用传参就是无效的，会被忽略。

### 其他方法

- return：将 yield 表达式替换成一个 return 语句，并立即结束当前迭代

```js
gen.return(2); // {value: 2, done: true}
```

- throw：将 yield 表达式替换成一个 throw 语句，也是立即结束，同时会抛出一个错误

```js
gen.throw(new Error("出错了")); // Uncaught Error: 出错了
```

- yield\*：在 generator 函数中调用另一个 generator 函数。调用效果和普通函数相同。

```js
function* gen() {
  yield 1;
  yield* gen2();
  yield 4;
  return 5;
}
function* gen2() {
  yield 2;
  return 3;
}
```

如果只是用 yield 执行，则会视作是执行普通函数，并且此 yield 的返回值是一个迭代对象，就像直接执行一样。

## 异步 Generator

异步 Generator 其实是协程在 js 中的实现。

> 协程：
> 意思是多个线程互相协作，完成异步任务。
> 协程有点像函数，又有点像线程。它的运行流程大致如下。
>
> - 第一步，协程 A 开始执行。
> - 第二步，协程 A 执行到一半，进入暂停，执行权转移到协程 B。
> - 第三步，（一段时间后）协程 B 交还执行权。
> - 第四步，协程 A 恢复执行。
> 
> 上面流程的协程 A，就是异步任务，因为它分成两段（或多段）执行。

比如：

```js
function* asyncJob() {
  // ...其他代码
  var f = yield readFile(fileA);
  // ...其他代码
}
```

其中的 yield 命令表示执行到此处，执行权将交给其他协程。也就是说，yield 命令是异步两个阶段的分界线。协程遇到 yield 命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点，就是代码的写法非常像同步操作。

并且，generator 还有自己的处理错误机制（throw）和从函数外改变函数内上下文的能力（next 函数传参改变内部变量），因此非常适合封装异步任务。

下面是一个简单的例子：

```js
const fetch = require("node-fetch");

function* gen() {
  const url = "https://api.github.com/users/github";
  const result = yield fetch(url);
  console.log(result.bio);
}

const g = gen();
const result = g.next();

result.value
  .then(function (data) {
    return data.json();
  })
  .then(function (data) {
    g.next(data);
  });
```

实际上，async/await 就是对这种形式的改进，async 就是对 generator 执行的自动化的封装。
