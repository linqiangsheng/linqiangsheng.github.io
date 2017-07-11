---
title: Debug with Node.js V8
---
简介了不同Node.js版本，如何使用V8引擎调试Node.js端代码的方案。若存在复杂的需要调试运行的Node.js端代码，建议将node升级到v6.3.0版本或更高版本。其推出了inspect指令，可通过chrome dev utils进行代码调试。

### 一、指令
```javascript
debug //例如 node debug app.js
--debug-brk
--inspect // >= 6.3.0 启动浏览器调试Node代码
--inspect-brk // >= 7.7.0 启动inspect之后停止于第一行代码
```
另外，在v8.0.0之后，node推出了inspector对象。用于与inspect的流程相辅相成，该对象目前属于试验性阶段，本文不做过渡讲解，有兴趣的同学可以到官方api上进行了解尝试。

#### debug
执行node debug app.js后，程序将进入debug模式。由于debug不存在界面操作，因此需要了解一下常用debug命令，方便进行调试跟踪。
``` javascript
c //cont的简写，继续执行
n //next的简写，跳到下一步
s //step的简写，跳到内部
o //out的简写，调出当前栈
pause //暂停运行

sb(1) //在当前文件第一行设置断点
sb('fn()') //在函数fn()内设置断点
sb('script.js', 1) //在script.js文件的第一行设置断点，若文件未被加载，会抛出警告，请勿惊慌，属于正常现象。
cb('script.js', 1) //删除script.js文件的第一行的断点

bt //打出目前执行的堆栈
list(5) //打印出当前代码断点位置的前后5行代码
watch(expr) //监控
unwatch(expr) //取消监控
watchers //打印出所有监控
repl //进入可执行命令行输入环境

run //启动
restart //重启
kill //关闭
```

#### Inspect
讲真，debug模式在调试上效率并不高。Node在v6.3.0之后推出了inspect模式，与chrome调试工具协作，可以像我们平常调试前端代码一样调试后台的node.js代码。
``` javascript
node --inspect app.js
```
执行以上命令后，控制台会返回chrome访问的地址，打开即可。
若你需要调试的代码并不存在任何服务，需要在启动调试时停住在第一行代码，方便你添加断点或监控其他数据，那么这时候你可以执行以下命令：
``` javascript
node --debug-brk --inspect app.js
```

#### inspect-brk
v7.7.0之后，node追加了inspect-brk标记，用于替代--debug-brk --inspect。于是，你可以直接执行以下命令即可：
``` javascript
node --inspect-brk app.js
```

### 二、Node V4.6.0调试
由于v4.*版本并不默认支持inspect方式，目前实践可用的调试方式有以下几个方案：
1. webstorm 强大的IDEA，可以支持nodejs的debug模式
2. 使用node-inspector模块，实现类似于inspect的调试方案
3. console.log，控制台打印日志，逐步分析问题
4. 终极方案，升级Node到6.3.0及以上
