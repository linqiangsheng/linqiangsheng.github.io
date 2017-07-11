---
title: Mocha
---
纠正发音，“摩卡”。Mocha是一个测试框架，就是运行测试的工具。简单的说，它就是用来运行测试脚本的。
测试脚本，用于测试源码的脚本，通常的做法是在被测试文件后添加.test后缀。例如add.js的测试脚本是add.test.js。
每个测试脚本内需要有一个或以上“测试套件”，每个套件需要有一个或以上“测试用例”。每个用例需要有一个或以上“断言”。

### Mocha
#### 一、Mocha是什么？
正如上述，Mocha是一个测试框架。
#### 二、Mocha的基础用法
1. 指定文件执行
``` javascript
$ mocha add.test.js
```
2. 同时执行多个文件或文件目录
``` javascript
$ mocha add.test.js testFolder
```
3. 执行一级子目录test文件夹下的测试脚本
``` javascript
$ mocha
```
4. 遍历执行子目录test文件夹下的测试脚本
``` javascript
$ mocha --recursive
```
5. 通配符，同时指定多个文件
``` javascript
$ mocha spec/{my, awesome}.js // 执行specmy.js和specawesome.js
$ mocha test/unit/*.js //执行test/unit下的所有js文件
$ mocha 'test/**/*.@(js|jsx)' //执行test目录下所有js，jsx后缀的文件
```
#### 三、常用命令行参数
1. --help, -h 帮助
2. -reporter, -R 输出日志格式，默认是spec格式，官方提供了其他[格式报告](http://mochajs.org/#reporters)
3. --growl, -G 测试结果在桌面显示
4. --watch, -w 监测脚本变化，自动运行
5. --bail, -b 只要有一个测试用例没有通过就停止执行
6. --grep, g 搜索测试用例名称，并只执行搜索结构的测试用例
7. --invert, -i 只执行--grep之外的测试用例，需要与--grep共用
8. --timeout-t 执行的超时等待时长，默认是2000毫秒
9. -s 高亮显示定义的超时用例，例如-s 1000

#### 四、配置文件mocha.opts
Mocha运行在test目录下配置mocha.opts文件。把命令行参数写在里面，可简化执行时的脚本。例如
``` javascript
$ mocha --report spec --grep "test one" --invert
```
将
``` javascript
test //该配置指明测试用例存放的文件夹，默认是test，可修改为其他目录
--report spec
--grep "test one"
--invert
```
写入mocha.opts里面，然后执行
``` javascript
$mocha
```
与前面的脚本执行结果是一样的。

#### 五、ES6测试
如果测试脚本是由ES6写的，那么需要进行Bable转码。
``` javascript
$ npm install babel-core babel-preset-es2015 --save-dev
```
然后在项目目录下新建.babelrc配置文件
```javascript
{
	"presets": ["es2015"]
}
```
最后，使用--compilers参数指定测试脚本的转码器
```javascript
$ mocha --compilers js:babel-core/register //冒号左边js是文件后缀名。右边是处理这一类文件的模块名。
```
Babel默认不会对Iterator、Generator、Promise、Map、Set等全局对象以及一些全局对象的方法进行转码。如果需要，需要安装babel-polyfill，并在脚本的头部执行引入。

#### 六、异步测试
Mocha内部支持Promise
```javascript
it('异步请求', function(){
	return fetch('https://api.com')
		.then(function(res) {
			return res.json();
		}).then(function(json) {
			expect(json).to.be.an('object');
		})
})
```

#### 七、钩子
Mocha允许在describe中，使用before(), after(), beforeEach(), afterEach()四个钩子。

#### 八、测试用例管理
only && skip方法。例如在测试脚本中，
``` javascript
it.only('A', function() {
	expect(add(1, 1).to.be.equal(2));
});
it('B', function() {
	expect(add(1, 2).to.be.equal(3));
});
```
只有A用例会被执行，skip则是相反。