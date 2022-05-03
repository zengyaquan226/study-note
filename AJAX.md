# AJAX

## http协议（超文本传输协议）

### 请求报文

请求报文分成：

1. 请求行
2. 请求头
3. 请求空行
4. 请求体

### 响应报文

响应报文分成：

1. 响应行
2. 响应头
3. 响应空行
4. 响应体



## 原生AJAX

### 模拟的服务器

**可以不需要管**

```js
// 1. 引入express
const express = require('express');

// 2. 创建应用对象
const app = express();

// 3. 创建路由规则
app.get('/server', (request, response) => {
  // 设置响应头 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 设置响应体
  response.send("Hello Ajax");
});

// 4. 监听服务
app.listen(8000, () => {
  console.log("服务已经启动, 8000 端口监听中...");
 })

```



### 发送get方式的请求

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Ajax GET 请求</title>
  <style>
    #result {
      width: 200px;
      height: 100px;
      border: solid 1px #90b;
    }
  </style>
</head>
<body>
  <button>点击发送请求</button>
  <div id="result"></div>
  <script>
    // 获取button元素
    const btn = document.getElementsByTagName('button')[0];
    // 绑定到div上
    const resToDiv = document.getElementById('result');
    // 绑定事件
    btn.onclick = function() {
      // 打印测试
      // console.log('test');
      // 1 创建对象 XMLHttpRequest代表 ajax
      const xhr = new XMLHttpRequest();
      // 2 初始化 设置请求方式和URL (打开)
      xhr.open("GET", 'http://127.0.0.1:8000/server');
      // 3 发送 (send)
      xhr.send();
      // 4 事件绑定 处理服务端返回的结果
      // onreadystatechange 当xhr对象的 readystate 属性进行改变的时候的回调方法
      // readystate 五个值
      // 0-未初始化 1-open完毕 2-send完毕 3-服务器部分消息返回 4-服务器所有消息返回
      xhr.onreadystatechange = function() {
        // 进行判断 服务器端返回的值
        if(xhr.readyState === 4) {
          // 判断响应状态码 当 状态码在200-300表示正常
          if(xhr.status >= 200 && xhr.status < 300) {
            // 服务器返回的响应报文
            // console.log(xhr.status);// 响应行的状态码
            // console.log(xhr.statusText);// 响应行的状态字符串
            // console.log(xhr.getAllResponseHeaders());// 响应头的所有信息
            // console.log(xhr.response);// 响应体的所有内容
          
            // 将响应体装入div中
            resToDiv.innerHTML = xhr.response;

          }
        }
      }
    }
  </script>
</body>
</html>

```



### 发送Post方式的请求

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Ajax POST 请求</title>
  <style>
    #result {
      width: 200px;
      height: 100px;
      border: solid 1px #90b;
    }
  </style>
</head>
<body>
  <div id="result"></div>
  <script>
      // 获取元素对象
      const result = document.getElementById("result");
      // 绑定事件 鼠标离开的事件 将调用这个回调方法
      result.addEventListener("mouseover", function(){
          // console.log("test");
          // 1 创建对象
          const xhr = new XMLHttpRequest();
          // 2 初始化
          xhr.open("POST", 'http://127.0.0.1:8000/server');
          // 设置请求体内容的类型
          xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
          // 自定义头信息 需要在服务器的方向进行添加对应的响应头信息
          xhr.setRequestHeader('name','hello');
          // 3 发送 信息 
          // 这里的发送消息虽然可以任意的方式进行发送 最好发服务器可以解析的
          xhr.send('a=10&b=20');
          // 处理服务器返回的结果
          xhr.onreadystatechange = function() {
              if(xhr.readyState === 4) {
                  if(xhr.status >= 200 && xhr.status < 300) {
                      // 将响应体的内容写入div中
                      result.innerHTML = xhr.response;
                  }
              }
          }
      });
  </script>
</body>
</html>

```

**对应的服务器**

```js
// 1. 引入express
const express = require('express');

// 2. 创建应用对象
const app = express();

// 3. 创建路由规则 处理 get 请求
app.get('/server', (request, response) => {
  // 设置响应头 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 设置响应体
  response.send("Hello Ajax");
});


// 3. 创建路由规则 处理 全部 请求
app.all('/server', (request, response) => { 
  // 设置响应头, 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 请求可以接受自定义的请求头 不需要是预先准备好的
  response.setHeader('Access-Control-Allow-Headers','*');
  // 设置响应体
  response.send("Hello Ajax POST");
});

// 4. 监听服务
app.listen(8000, () => {
  console.log("服务已经启动, 8000 端口监听中...");
 })

```

### 服务器返回json类型的数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- <meta http-equiv="X-UA-Compatible" content="IE=edge"> -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Ajax GET 请求</title>
  <style>
    #result {
      width: 200px;
      height: 100px;
      border: solid 1px #90b;
    }
  </style>
</head>
<body>
  <div id="result"></div>
  <script>
      const result = document.getElementById('result');
      // 绑定键盘点击事件
      window.onkeydown = function() {
          // console.log('test');
          // 创建对象
          const xhr = new XMLHttpRequest();
          // 对服务器返回的数据进行自动转换成json的格式
          xhr.responseType = 'json';
          // 初始化 请求方式 url
          xhr.open('GET', 'http://127.0.0.1:8000/json-server');
          // 发送消息
          xhr.send();
          // 处理服务器返回的结果
          xhr.onreadystatechange = function() {
              if(xhr.readyState === 4) {
                  if(xhr.status >= 200 && xhr.status < 300) {
                      // 处理返回的结果
                      // result.innerHTML = xhr.response;
                      // 1 当服务器返回的是json的格式的数据时，进行手动转换
                      // let data = JSON.parse(xhr.response);
                      // console.log(data);
                      // 取出json中为name的值
                      // result.innerHTML = data.name;
                      // 2 json对象的自动转换
                      console.log(xhr.response);
                      result.innerHTML = xhr.response.name;
                  }
              }
          }
      }
  </script>
</body>
</html>

```



**对应的服务器**

```js
// 1. 引入express
const express = require('express');

// 2. 创建应用对象
const app = express();

// 3. 创建路由规则 处理 get 请求
app.get('/server', (request, response) => {
  // 设置响应头 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 设置响应体
  response.send("Hello Ajax");
});


// 3. 创建路由规则 处理 全部 请求
app.all('/server', (request, response) => { 
  // 设置响应头, 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 请求可以接受自定义的请求头 不需要是预先准备好的
  response.setHeader('Access-Control-Allow-Headers','*');
  // 设置响应体
  response.send("Hello Ajax POST");
});

// 3. 创建路由规则 处理 全部 请求
app.all('/json-server', (request, response) => { 
  // 设置响应头, 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 请求可以接受自定义的请求头 不需要是预先准备好的
  response.setHeader('Access-Control-Allow-Headers','*');
  // 返回的数据
  const data = {
    name: '张三'
  };
  // 将data转成字符串
  let str = JSON.stringify(data);
  // 设置响应体
  response.send(str);
});

// 4. 监听服务
app.listen(8000, () => {
  console.log("服务已经启动, 8000 端口监听中...");
 })

```



### 解决IE缓存问题

**只需要每次发请求的时候带上一个时间戳，每次的请求都不一样即可**





### 网络延迟和网络异常

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- <meta http-equiv="X-UA-Compatible" content="IE=edge"> -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Ajax GET 请求</title>
  <style>
    #result {
      width: 200px;
      height: 100px;
      border: solid 1px #90b;
    }
  </style>
</head>
<body>
  <button>点击发送请求</button>
  <div id="result"></div>
  <script>
      const btn = document.getElementsByTagName('button')[0];
      const result = document.getElementById('result');
      // 绑定点击事件
      btn.addEventListener('click', function() {
          // console.log('test');
          const xhr = new XMLHttpRequest();
          // 设置超时时间 当时间过来将不在发起请求
          xhr.timeout = 2000;
          // 请求超时的回调方法
          xhr.ontimeout = function() {
              alert("网络延迟，请稍后再试");
          };
          // 网络异常回调
          xhr.onerror = function() {
              alert("您的网络走小差了");
          };
          xhr.open('GET', 'http://127.0.0.1:8000/delay');
          xhr.send();
          xhr.onreadystatechange = function() {
              if(xhr.readyState === 4) {
                  if(xhr.status >= 200 && xhr.status < 300) {
                      result.innerHTML = xhr.response;
                  }
              }
          }
      });
  </script>
</body>
</html>

```



**对应的服务器添加的**

```js
// 3. 创建路由规则 处理 get 请求
app.get('/delay', (request, response) => {
  // 设置响应头 设置允许跨域
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 设置响应体 设置3秒后进行发送
  setTimeout(() => {
    response.send("请求超时");
  }, 3000);
});
```



### 取消发送请求

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- <meta http-equiv="X-UA-Compatible" content="IE=edge"> -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>Ajax GET 请求</title>
  <style>
    #result {
      width: 200px;
      height: 100px;
      border: solid 1px #90b;
    }
  </style>
</head>
<body>
  <button>点击发送请求</button>
  <button>点击取消请求</button>
  <div id="result"></div>
  <script>
      // 获取元素对象 将button标签的 放到数组中
      const btns = document.querySelectorAll('button');
      // 扩大作用域
      let xhr = null;
      // 将第一个按钮添加绑定事件
      btns[0].onclick = function() {
          xhr = new XMLHttpRequest();
          xhr.open('GET', 'http://127.0.0.1:8000/delay');
          xhr.send();
      }
      // 第二个按钮进行绑定
      btns[1].onclick = function() {
          // 手动取消发送请求
          xhr.abort();
      }
  </script>
</body>
</html>

```



### 重复请求问题

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <!-- <meta http-equiv="X-UA-Compatible" content="IE=edge"> -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <title>重复请求问题</title>
  <style>
    #result {
      width: 200px;
      height: 100px;
      border: solid 1px #90b;
    }
  </style>
</head>
<body>
  <button>点击发送请求</button>
  <div id="result"></div>
  <script>
      // 获取元素对象 将button标签的 放到数组中
      const btns = document.querySelectorAll('button');
      // 加一个标识符来判断重复请求问题
      let isSending = false; // false 表示未进行发送 true 表示正在发送
      // 扩大作用域
      let xhr = null;
      // 将第一个按钮添加绑定事件
      btns[0].onclick = function() {
          // 当标识符未true表示之前的请求还没有结束 将其手动结束
          if(isSending) xhr.abort();
          xhr = new XMLHttpRequest();
          // 正在准备发送修改 isSending 的值
          isSending = true;
          xhr.open('GET', 'http://127.0.0.1:8000/delay');
          xhr.send();
          xhr.onreadystatechange = function() {
            if(xhr.readyState === 4) {
              // 表示服务器的响应都已经结束 无论是否正确
              // 修改值
              isSending = false;
            }
          }
      }
  </script>
</body>
</html>

```



## JQuery中AJAX

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>jQuery 发送 AJAX 请求</title>
    <link crossorigin="anonymous" href="https://cdn.bootcss.com/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
    <script crossorigin="anonymous" src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
</head>
<body>
    <div class="container">
        <h2 class="page-header">jQuery发送AJAX请求 </h2>
        <button class="btn btn-primary">GET</button>
        <button class="btn btn-danger">POST</button>
        <button class="btn btn-info">通用型方法ajax</button>
    </div>
    <script>
        // 对这个按钮进行绑定 通过jquery的方式
        $('button').eq(0).click(function() {
            $.get('http://127.0.0.1:8000/jquery', {a:100,b:200},
            function(data) {
                // data就是服务器返回的响应体
                // json 是指定服务器返回的类型是json类型的
                console.log(data);
            }, 'json');
        });

        // 对这个按钮进行绑定 通过jquery的方式
        $('button').eq(1).click(function() {
            $.post('http://127.0.0.1:8000/jquery', {a:100,b:200},
            function(data) {
                // data就是服务器返回的响应体
                // json 是指定服务器返回的类型是json类型的
                console.log(data);
            });
        });

        // 写一个通用的ajax
        $('button').eq(2).click(function() {
            $.ajax({
                // url
                url: 'http://127.0.0.1:8000/jquery',
                // data数据
                data: {a:100, b:200},
                // 请求类型
                type: 'GET',
                // 服务器返回数据的类型
                dataType: 'json',
                // 成功的回调方法 data就是服务器返回的响应体
                success: function(data) {
                    console.log(data);
                },
                // 设置超时等待 过了2s将取消请求
                timeout: 2000,
                // 出现的回调方法
                error: function() {
                    console.log('出错了');
                },
                headers: {
                    // 自定义的请求头需要在服务器上加上
                    // 请求可以接受自定义的请求头 不需要是预先准备好的
                    // response.setHeader('Access-Control-Allow-Headers','*');
                    hello: 'world'
                }
            });
        });
    </script>
</body>
</html>

```



## 通过axios发送ajax请求

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>axios 发送 AJAX请求</title>
    <script crossorigin="anonymous" src="https://cdn.bootcdn.net/ajax/libs/axios/0.19.2/axios.js"></script>
</head>

<body>
    <button>GET</button>
    <button>POST</button>
    <button>AJAX</button>

    <script>
        let btns = document.querySelectorAll('button');
        //配置baseURL
        axios.defaults.baseURL = "http://127.0.0.1:8000";
        btns[0].onclick = function() {
            axios.get('/axios', {
                // 设置url的参数
                params: {
                    id: 100,
                    name: '张三'
                },
                // 设置请求头的信息
                headers: {
                    age: 20
                }
            }).then(value => {
                console.log(value);
                console.log(value.data.name);
            })
        }

        // 发送post请求
        btns[1].onclick = function() {
            axios.post('/axios', {
                // 这里是请求体的内容
                // 以json的格式
                username: 'admin',
                password: 'admin'
            }, {
                // 设置url的参数
                params: {
                    id: 100,
                    name: '张三'
                },
                // 设置请求头的信息
                headers: {
                    age: 20
                }
            }).then(value => {
                console.log(value);
                console.log(value.data.name);
            })
        }
   
        btns[2].onclick = function() {
            axios({
                // 请求方式
                method: 'POST',
                // url
                url: '/axios',
                // url上的参数
                params: {
                    a: 100,
                    b: 200
                },
                // 请求头信息
                headers: {
                    high: 200
                },
                // 请求体信息
                data: {
                    user: 'admin',
                    pwd: 'admin'
                }
            }).then(value => {
                // 对服务器返回的信息进行响应
                console.log(value);
            });
        }
   </script>
</body>

</html>

```



## 通过fetch发送请求

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>fetch 发送 AJAX请求</title>
</head>
<body>
    <button>AJAX请求</button>
    <script>
        const btn = document.querySelector('button');
        btn.onclick = function(){
            fetch('http://127.0.0.1:8000/fetch?vip=10', {
                //请求方法
                method: 'POST',
                //请求头
                headers: {
                    name:'atguigu'
                },
                //请求体
                body: 'username=admin&password=admin'
            }).then(response => {
                // return response.text();
                return response.json();
            }).then(response=>{
                console.log(response);
            });
        }
    </script>
</body>
</html>

```



## 跨域

**ajax默认只可以进行同源的进行通信，即数据和页面都是都是同一个服务器**



### 通过jsonp解决跨域问题

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JSONP实践</title>
</head>

<body>
    用户名<input type="text" id="username"></input>
    <!--用于显示当用户已经存在的时候做显示-->
    <p></p>
    <script>
        // 获取input元素
        const input = document.querySelector('input');
        // 获取p标签元素
        const p = document.querySelector('p');
        // 写一个处理方法
        function handle(data) {
            // 将input的边框改成红色
            input.style.border = "solid 1px #f00";
            // 修改p标签的提示文本
            p.innerHTML = data.msg;
        }
        // 绑定 当焦点失去的时候触发
        input.onblur = function () {
            // 获取用户输入的值
            let username = this.value;
            //向服务器端发送请求 检测用户名是否存在
            //1.创建script标签
            const script = document.createElement('script');
            //2.设置标签的src属性
            script.src = "http://127.0.0.1:8000/check";
            //3.将script插入到文档中
            document.body.appendChild(script);
        }
    </script>
</body>

</html>
```



**服务器代码**

```js
//用户名检测是否存在
app.all('/check',(request, response) => {
  // response.send('console.log("hello jsonp")');
  const data = {
      exist: 1,
      msg: '用户名已经存在'
  };
  //将数据转化为字符串
  let str = JSON.stringify(data);
  //返回结果
  response.end(`handle(${str})`);
});
```



**这个就是利用script的本身自带的跨域的功能，服务器直接放回一个js的代码，放到script进行解析即可**



### 通过jquery来解决跨域问题

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>jQuery-jsonp</title>
    <style>
        #result{
            width:300px;
            height:100px;
            border:solid 1px #089;
        }
    </style>
    <script crossorigin="anonymous" src='https://cdn.bootcss.com/jquery/3.5.0/jquery.min.js'></script>
</head>
<body>
    <button>点击发送 jsonp 请求</button>
    <div id="result">

    </div>
    <script>
        $('button').eq(0).click(function(){
            $.getJSON('http://127.0.0.1:8000/jquery-jsonp?callback=?', function(data){
                $('#result').html(`
                    名称: ${data.name}<br>
                    校区: ${data.city}
                `)
            });
        });
    </script>
</body>
</html>

```

**服务器代码**

```js
app.all('/jquery-jsonp',(request, response) => {
    // response.send('console.log("hello jsonp")');
    const data = {
        name:'尚硅谷',
        city: ['北京','上海','深圳']
    };
    //将数据转化为字符串
    let str = JSON.stringify(data);
    //接收 callback 参数
    let cb = request.query.callback;

    //返回结果
    response.end(`${cb}(${str})`);
});
```



### 通过cors来解决跨域问题

**由服务器进行配置，设置一些响应头来告诉浏览器，该请求允许跨域，浏览器收到该响应以后就会对响应执行**

```js
app.all('/cors-server', (request, response)=>{
    //设置响应头
    // 这个是允许所有的请求地址进行跨域
    response.setHeader("Access-Control-Allow-Origin", "*");
    // 这个是允许自己定义任何形式的请求头
    response.setHeader("Access-Control-Allow-Headers", '*');
    // 这个是允许以任何的方式进行请求 （默认是get post）
    response.setHeader("Access-Control-Allow-Method", '*');
    // response.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:5500");
    response.send('hello CORS');
});
```

