### Webpack in dev with express

---
title: Webpack-dev-middleware fileSystem express
---

本文预设各位读者对webpack在node中的中间件引入有一定了解。

#### 需求
1. 使用webpack的html-webpack-plugin插件，根据指定的模板文件，生成目标html文件。
2. 服务端使用express（或其他框架）。
3. 开发模式下不需要webpack打包出任何存在于物理磁盘的文件。

#### 背景
使用express + webpack，开发模式下，常用的做法是将webpack作为中间件引入到express中，并将webpack打包构建的结果写入内存（不输出到物理磁盘上，减少io消耗，提高性能）。
那么问题来了，express的render方法，是通过判断传入参数，例如说index，在指定的views文件夹下面，是否有该参数名的html或ejs等模板文件。也就是说，render会有依赖于判断文件是否存在的条件。但webpack开发模式下并不会生成真实存在的文件。该如何解决这个问题呢？

#### 方案
一、方案一
区分开发、生产、测试等环境，每个环境使用自己的html模板，并在render之前进行环境判断，切换不同的html代码。

二、方案二
既然express整合了webpack的中间件，那么能不能把webpack-dev服务生成的，存在内存中的数据读出来，通过express是render方法渲染到前端呢？答案是肯定的。
就此思路，查了下webpack-dev-middleware的源码。发现模块export的内容里面，存在fileSystem对象，并且该对象具备readFileSync方法。那么上代码吧。
```javascript
var ejs = require('ejs');
var devMiddleware = webpackDevMiddleware(compiler, {
	noInfo: true,
		publicPath: webpackConfig.output.publicPath,
		stats: {
		    colors: true
		}
	});

app.use(devMiddleware);
express.response.render = function(name, options) {
	...
	var htmlFilePath = '../views/index.html';
	this.write(ejs.render(devMiddleware.fileSystem.readFileSync(htmlFilePath, 'utf-8')));
	this.end();
}
```
