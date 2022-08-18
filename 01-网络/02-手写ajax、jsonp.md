# 实战应用

# 原生 js 发送 Ajax

    <body><divid="test"></div><script>varxhr=null;
    // 1.创建一个http请求的对象，readyState 0if (window.XMLHttpRequest) {
    xhr=newXMLHttpRequest();
            } else {
    xhr=newActiveXObject("Microsoft.XMLHttp");
            }
    console.log(xhr.readyState);
    // 2. 初始化服务连接，建立与服务器之间的连接。 readyState 1// 如果open第三个参数传true，或者不传，为异步模式。如果传false，为同步模式。xhr.open("get", "http://developer.duyiedu.com/edu/testAjaxCrossOrigin", false);
    console.log(xhr.readyState);
    // 3. 注意： 发送前，先设置方法xhr.onreadystatechange=function () {
    console.log(xhr.readyState);            // 以此打印：2 3 4 //readyState == 4表示请求完成，已经接收到数据。//status == 200  网络请求，结果都会有一个状态码。来表示这个请求是否正常if (xhr.readyState==4&&xhr.status==200) {
    console.log(xhr.responseText);
    document.getElementById("test").innerText=xhr.responseText;
    vardata=JSON.parse(xhr.responseText);
    console.log(data);
                }
            }
    // 4. 发送   readyState 1xhr.send();
    console.log('++++++++++++++');
    </script></body>

- open 第三个参数传 false，为同步模式

**![image.png](../.gitbook/assests/1596079073040-588a7135-8b03-4cc8-a134-a79bb6e66fd3.png)**

- open 第三个参数传 true，或者不传，为异步模式 ![image.png](../.gitbook/assests/1596079121698-8d0ecd8d-b494-4e82-baf4-40d362307ab1.png)

\*\*
\*\*

**http 状态码：**

HTTP 状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。

HTTP 状态码共分为 5 种类型：

分类分类描述 1**信息，服务器收到请求，需要请求者继续执行操作 2**成功，操作被成功接收并处理 3**重定向，需要进一步的操作以完成请求 4**客户端错误，请求包含语法错误或无法完成请求 5\*\*服务器错误，服务器在处理请求的过程中发生了错误

# JQuery 发送 jsonp

**jsonp 格式哪里特殊？**

> 发送的时候，会带上一个参数 callback

> 返回的结果不是 json，是 ​ `callback(json);`

**jsonp 跨域，只能使用 get 方法!**

> jsonp 跨域，只能使用 get 方法，如果我们设置的是 post 方法，jquery 会自动转为 get 方法。

**是不是在 jsonp 里面我只能使用 get 方法？是不是我设置的 post 方法都会转为 get 方法？**

> 不是。
>
> jquery 会先判断是否同源，如果同源，那么设置的是 get 就是 get，设置的 post 就是 post。
>
> 如果不是同源（跨域），无论设置的什么，都改为 get。

    <head><metacharset="UTF-8"><title>Title</title><scriptsrc="jquery.js"></script><script>//jsonp格式哪里特殊？//发送的时候，会带上一个参数callback//返回的结果不是json//callback的名 + ( + json + );$.ajax({
    url: "http://developer.duyiedu.com/edu/testJsonp",
    type: "post",
    dataType: "jsonp",
    success: function (data) {
    console.log(data);
                }
            });
    </script></head>

[📎jquery.js](https://www.yuque.com/attachments/yuque/0/2020/js/714097/1596082748046-e67f1f16-5b2c-4545-a769-ac87873ff896.js)

# jsonP 原理

**JSONP 原理：**

- 前端部分

1.  判断请求与当前页面的域，是否同源，如果同源则发送正常的 ajax，就没有跨域的事情了。
2.  如果不同源，生成一个 script 标签
3.  生成一个随机的 callback 名字，需要自己提前**创建一个名为这个\*\***callba\***\*ck\*\***名字\***\*的方法**。
4.  设置 script 标签的 src，设置为要请求的接口。
5.  将 callback 作为参数拼接在后面。

- 后端部分

1. 后端接收到请求后，开始准备要返回的数据
2. 后端拼接数据，将要返回的数据用 callback 的值和括号包裹起来

> 例如：callback=test，要返回的数据为{"a":1, "b":2},
>
> 就要拼接为：test({"a":1, "b":2});

1.  将内容返回。

- 浏览器部分

1. 浏览器接收到内容，会当做 js 代码来执行。
2. 从而执行名为 test 的方法。这样我们就接收到了后端返回给我们的对象。

**案例：**

    <body><script>functiontest(data) {
    console.log('接口测试：',data);
            }
    </script><!--因为我想从一个接口获取一个数据但是这个接口和当前页面不是同源的。（也就是跨域了）但是这个接口是支持JSONP的。script标签，有src属性，所以可以发出网络请求script标签，虽然可以引用其他域的资源，浏览器不限制。但是，浏览器会将返回的内容，作为js代码执行。test({"status":"ok","msg":"Hello! There is DuYi education!"})
    相当于调用了test方法，传入了一个json对象作为参数。--><scriptsrc='http://developer.duyiedu.com/edu/testJsonp?callback=test'></script></body>

![image.png](../.gitbook/assests/1596105176756-39bdb4a3-fabb-4dd0-83a1-3f2b86d45d59.png)

**jsonP 原理实现:**

    <body><script>var$= {
    ajax: function (options) {
    varurl=options.url;
    vartype=options.type;
    vardataType=options.dataType;
    //1. 判断是否同源（协议，域名，端口号）//获取目标url的域vartargetProtocol="";        //目标接口的协议vartargetHost="";            //目标接口的host，host是包涵域名和端口的//2. 如果url不带http，那么访问的一定是相对路径，相对路径一定是同源的。if (url.indexOf("http://") ==0||url.indexOf("https://") ==0) {
    vartargetUrl=newURL(url);
    targetProtocol=targetUrl.protocol;
    targetHost=targetUrl.host;
                    } else {
    targetProtocol=location.protocol;
    targetHost=location.host;
                    }
    // 3. 首先判断是否为jsonp，因为不是jsonp不用做其他的判断，直接发送ajaxif (dataType=="jsonp") {
    // 3.1 要看是否同源if (location.protocol==targetProtocol&&location.host==targetHost) {
    console.log('同源');              //此处省略。因为同源，jsonp会当做普通的ajax做请求                    } else {
    // 跨域，随机生成一个callbackvarcallback="cb"+Math.floor(Math.random() *1000000);
    window[callback] =options.success;                 // 给window上添加一个callback方法varscript=document.createElement("script");      // 生成script标签。// script.src 拼接url + callback ; http://developer.duyiedu.com/edu/testJsonp?callback=if (url.indexOf("?") >0) {                         //表示有参数script.src=url+"&callback="+callback;
                            } else {                                            //表示没有参数script.src=url+"?callback="+callback;
                            }
    script.id=callback;
    document.head.appendChild(script);
                        }
                    }
                }
            }
    // 调用$.ajax({
    url: "http://developer.duyiedu.com/edu/testJsonp",
    type: "get",
    dataType: "jsonp",
    success: function (data) {
    console.log('请求成功：',data);
                }
            });
    </script></body>

![image.png](../.gitbook/assests/1596110659679-61fbd09b-3a77-4f43-8f94-c60c42fd16ee.png)
