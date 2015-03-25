---
layout: default
title: Ajax总结
comments: true
---

         
### 1. 基本概念及原理

AJAX是asynchronrous javascript and xml(异步javascript和xml)。
其本质是利用浏览器内置的XMLHttpRequest对象异步向服务器发送请求，并且利用服务器返回的数据来部分更新当前页面。
AJAX技术的优点有：

* 加载新内容无需刷新整个页面
* 不打断用户的操作，用户的体验好
* 按需获取数据，浏览器与服务器之间数据的传输量减少
* 浏览器都支持，无需要安装插件
* 可以利用浏览器的计算能力

### 2. XMLHttpRequest对象的获取与属性

2.1 XMLHttpRequest对象的获取

实现XMLHttpRequest对象的技术并不是标准化的，不同厂商提供该对象的方式不一样，主要分为IE和非IE。

```javascript
  var xhr;
  if (window.XMLHttpRequest) {
    xhr = new XMLHttpRequest();
  } else { // IE5、IE6
    xhr = new ActiveXObject('Microsoft.XMLHTTP');
  }
```

2.2 方法：

`open(method, url, async)` 规定请求的类型、URL以及是否异步处理请求
        
  * `method` : 请求的类型；GET或POST
  * `url` : 文件中服务器上的位置
  * `async` : true（异步）或false（同步）

`setRequestHeader(header,value)` 向请求添加HTTP头

  * `header` : 规定头的名称
  * `value` : 规定头的值

`send(string)` 将请求发送到服务器

  * `string` : 仅用于POST请求


2.3 属性：

* `onreadystatechange` 绑定事件函数。当readyState值改变时，会触发onreadystatechange，执行绑定的函数。
* `readyState` XMLHttpRequest的状态。从0到4，分别表示：
  * 0 - 请求未初始化
  * 1 - 服务器连接已建立
  * 2 - 请求已接收
  * 3 - 请求处理中
  * 4 - 请求已经完成，且响应已就绪
* `status` 服务器响应状态
* `responseText` 获得字符串形式的响应数据
* `responseXML` 获得XML形式的响应数据

### 3. 示例

GET请求

```javascript
    function myFunction() {
      var xhr;
      if (window.XMLHttpRequest) {
        xhr = new XMLHttpRequest();
      } else { // IE5、IE6
        xhr = new ActiveXObject('Microsoft.XMLHTTP');
      }
      xhr.onreadystatechange = function {
        if (xhr.readyState==4) { // 只有readyState为4时，才能获得服务器返回的数据
          if (xhr.status==200) {
            document.getElementById('textHint').innerHTML=xhr.responseText;
          } else if (xhr.status==0) { // 本地响应成功
          	document.getElementById('resultDiv').innerHTML=xhr.responseText;
          }
        }
      }
      xhr.open('GET','a.txt',true); //a.txt文件在本地
      xhr.send();
    }
```

*源代码解释：*

* 当页面事件触发后执行myFunction时
* 创建XMLHttpRequest对象
* 为XMLHttpRequest对象绑定回调函数
* 将请求发送到服务器
* 服务器状态改变触发回调函数，将服务器返回的数据展现到页面上
* onreadystatechange 事件被触发 5 次（0 - 4），对应着 readyState 的每个变化。

POST请求

```javascript
    function myFunction2() {
      var xhr;
      if (window.XMLHttpRequest) {
        xhr = new XMLHttpRequest();
      } else { // IE5、IE6
        xhr = new ActiveXObject('Microsoft.XMLHTTP');
      }
      xhr.onreadystatechange = function {
        if (xhr.readyState==4) { // 只有readyState为4时，才能获得服务器返回的数据
          if (xhr.status==200) {
            document.getElementById('textHint').innerHTML=xhr.responseText;
          } else if (xhr.status==0) { // 本地响应成功
          	document.getElementById('resultDiv').innerHTML=xhr.responseText;
          }
        }
      }
      xhr.open('POST','ajax_test.do',true); 
      xhr.setRequestHeader("Content-type", "application/x-www-url-urlencoded");
      xhr.send("name=john&gendar=male");
    }
```

*源代码解释：*

* 当页面事件触发后执行myFunction时
* 创建XMLHttpRequest对象
* 为XMLHttpRequest对象绑定回调函数
* 将表单数据发送到服务器
* 服务器状态改变触发回调函数，将服务器返回的数据展现到页面上

*注意：按照http协议的要求，如果发送的POST请求，那么请求数据包里面应该有一个Content-type，但是XMLHttpRequest对象默认没有，所以需要调用setRequestHeader方法设置。*


GET与POST的区别

与POST相比，GET更简单也更快，并且在大部分情况下都能用。
然而，在以下情况下，请使用POST请求：

* 无法使用缓存文件（更新服务器上的文件或数据库）
* 向服务器发送大量数据（POST没有数据限制）
* 发送包含未知字符的用户输入时，POST比GET更稳定也更可靠



### 4. 注意问题


#### 异步、同步问题
  当AJAX对象向服务器发送同步请求时，浏览器不会向下执行，会等待服务器的响应。在完整的收到服务器返回的数据之前，浏览器处于锁定状态，用户不可进行操作。

####编码问题

#####  发送GET请求
  
  IE浏览器内置的AJAX对象，对中文参数使用GBK编码，而其它浏览器（FireFox、Chrome）使用UTF-8编码。
  服务器端默认使用ISO-8859-1去解码
  解决方案：
  
  1.让服务器对GET请求中的参数使用指定的编码格式进行解码。比如，对于Tomcat，可以修改conf/server.xml，只对GET方式有效
    
    <Connector port="8080" protocol="HTTP/1.1"
        connectionTimeout="2000"
        URIEncoding="UTF-8" (要加入的代码)
        redirectPort="8443" />
        
  2.对请求地址，使用encodeURI函数进行统一的编码（UTF-8）。
  
    xhr.open('GET',encodeURI(uri),true);
    
#####  发送POST请求
  
  如果发送POST请求，那么所有浏览器内置的AJAX对象都会对请求参数使用UTF-8进行编码，那么只要在服务器端设置：
  
    request.setCharacterEncoding("UTF-8");

####缓存问题

当使用IE浏览器时，如果使用GET方式发送请求，浏览器会先缓存之前访问的数据，如果访问的地址不变 ，不会向服务器发送请求。

解决方式1：使用POST方式发请求。

解决方式2：在请求地址后面加一个随机数