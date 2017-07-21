---
title: Webpack DllPlugin
---

#### 问题起源于
webpack在生产环境中打包特别慢。每个子项目平均要两到三分钟以上，在性能较差的电脑上甚至会达到10分钟左右，无法忍受！另外，打包出来的文件较大（基本都是1m+），影响加载速度，可重复利用性也较低。虽然说可以通过externals配置结合vendors导出公用模块，但该模块会收发布时的时间戳后缀影响，无疑更新版本后用户需要重新下载该模块。
基于以上种种问题，webpack模拟了c++类似的思想，推出了动态连接库DLLPlugin（使用别名的方式有兴趣的同学可以自己研究，最终实现的效果和这种模式实质上差不都）。
#### DLLPlugin将打包过程分成两步：

- **打包通用dll包**
- **引用dll包，打包业务代码**

在开始之前，简单介绍下webpack打包时使用方便调试使用的几个配置
```

-p #压缩
--display-modules #显示被隐藏的模块
--sort-modules-by size #结合display-modules 可以按大小排序查看那些依赖的模块较大，是否需要单独抽离
--profile #显示性能参数，查看耗时

```

#### **使用**DllPlugin
需要先新建一个dll.config.js的配置文件。假设说我们把react常用的相关模块进行抽离，那么配置文件如下：

```javascript

const webpack = require('webpack');

const vendors = [
	'react',
	'react-dom',
	'redux',
	//...... more
];

module.exports = {
	output: {
		path: 'public',
		filename: '[name].js',
		library: '[name]',
	},
	entry: {
		"react": vendors
	},
	plugins: [
		new webpack.DllPlugin({
			path: 'manifest/react.json', //生成的manifest文件，生产项目引用该文件以便于加载相关依赖库
			name: '[name]', //与entry的key一致，例如这里最后导出为react.js
			context: __dirname, //该路径名与生产项目配置一致
		}),
		
		//未配置该插件在加-p打包后页面会提示警告
		new webpack.DefinePlugin({
			'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'production')
		})
	],
};

```
保存该文件后，使用webpack进行打包：
```

webpack --config ./ddl.config.js -p

```

如果一切正常，会生成两个文件，**public/react.js** 以及 **manifest/react.json**。
#### 修改生产环境配置
这时候我们打开项目的打包配置文件webpack.prod.config.js，修改如下：
```javascript

module.exports = {
	plugins: [
		new webpack.DllReferencePlugin({
			context: __dirname,
			manifest: require('./manifest/react.json'),
		})
	]
}

```

至此，该工程打包的时候，都会跳过对dll.react.js内vendors声明的库进行编译，光是对react、react-dom的编译便可减少了一定时间。对简单的项目或依赖管理清晰的项目会有非常明显的打包速度提升！
**当然，切记要在你的html文件中引入public/react.js哦！**
#### 使用DllPlugin好处应该显而易见了：
- 提升打包效率
- 减小生产文件体积，抽离公共文件，提高页面加载效率
- 公共模块不会因为业务代码更新而需要重新加载，直接缓存cdn

#### 最后
简单阐述下配置过程中遇到的3个小问题。
1、单个dll配置无法导出多个manifest.json，目前并没有深究，只是使用了多个dll配置文件简单粗暴的解决了；
2、不要对antd进行打包，antd整包打包会引入超过2m（压缩后好像是600k以上）的体积增长。可通过配置.babelrc文件对antd的按需加载。
```json
{
  "presets": ["es2015", "react"],
  "plugins": ["antd"]
}
```
3、dev模式下也可以用哦，使用方法雷同，可以一定量的提升编译效率！
