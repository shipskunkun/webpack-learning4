## 5章 原理

### 5.1 webpack的流程

1. 初始化参数
2. 实例化 Compliler 对象，开始编译
3. 通过 entry 找入口文件
4. 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出
模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
5. 完成模块编译，每个模块被翻译了，而且依赖关系也确定好了
6. 组装成chunk
7. 输出完成


以上过程中， Webpack 会在特定的时间点广播特定的事件，插件在监听到感兴趣的
事件后会执行特定的逻辑，井且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

大概三个阶段

	初始化
	编译
	输出

初始化阶段就注册插件事件。

	1. 全局只有一个 compiler 实例，Compiler 责文件监听和启动编译。
	
	2. 加载插件，依次调用插件的 apply 方法，让插件可以监听后续的所有事件节点．同 时向插件传入 compiler实例的引用，以方便插件通过 compiler 调用 Webpack 供的 AP!

	
编译阶段
	
	Webpack 以开发模式运行时，每当检测到文件的变化，便有 次新的 Compilation 被创建
	Compilation 对象包含了当前的模块资源、编译生成资源、变化的文件等 。
	Compilation对象也提供了很多事件回调给插件进行扩展
	
	在编译阶段中，最重要的事件是 compilation ，因为在 compilation 阶段调用了 Loader,完成了每个模块的转换操作。
	
	
	
	
### 5.2 输出文件分析

bundle.j 文件 生成一个 modues, 所有安装股的模块都在里面	
根据 moduleId 找模块

### 5.3 编写 Loader

```	
module.exports = function (source ) { 
	// source compiler 传递给 Loader 的一个文件的原内容
	／／ 该函数需要返回处理后的内容，这里为了简单起见，直接将原内容返回了，相当于该Loader 有做任何转换
	re turn source ; 
};
```

由于 Loader 运行在 Node.js 中，所以我们可以调用任意 Node.js 自带的 API ，或者安装第三方模块进行调用：

```
const sass= require (’ node -sas s ’); 
module.exports = function (source) { 
	return sass (s ource) ;
}	
```

通过 loader-utils 获取 option 参数

```
const loaderUtils =require (’ loader-utils’); 
	module. exports = function (source) {. 
	／／获取用户为 Loader 传入的 options
	const options= loaderUtils.getOptions(this); 
	return source;
}
```
通过callback 返回loader 处理结果

参数：this.callback(err, content, sourceMap, AST )

```
module. exports = function (source) { 
	//通过 this .callba ck 告诉 Webpack 返回的结果
	this.callback(null, source, sourceMaps); 
	
	//当我们使用 this.callback 返回内容时 ，该 loader 必须返回 undefined,
	//以让 Webpack 知道该 Loader 返回的结果在 this.callback 中，而不是return中
	return;
```


如何使用本地的loader?
	
如果不，会怎样？确保loader 在node_modules 文件夹中		
需要将写好的loader发布到npm仓库，再安装到本地项目中使用

如果不发布呢？
	
使用 resolveLoader

```
module .exports = { 
resolveLoader:{ 
	／／去哪些目录下寻找 Loader ，有先后顺序之分
	modules :[’node modules ’,’. / loaders/ ’],
	}
}
```

编写一个简单的loader

```
function replace(source) { 
	／／使用正则表达式将 II @require ' .. lstylelindex.css ’转换成
	require ( ’../style lindex .css’); 
	return source.replace(l(\1\1 *@require) +((’|”).+(’|”)).*/, 
’ require($2 ) ;’); 

module. exports = function (content) { 
return replace(content);
};
```

### 5.4 编写 Plugin

Webpack 通过 Plugin 机制让其更灵活，以适应各种应用场景。在 Webpack 行的生命
周期中会广播许多事件， Plugin 可以监昕这些事件，在合适的时机通过 Webpack 提供的 API改变输出结果。


插件实例在获取到 compiler 对象后，就可以通过 compiler.plugin（事件名称 ，回调函数）监听到 Webpack 广播的事件，并且可以通过 compiler 对象去操作 Webpack


在开发 Plugin 时最常用的两个对象就是 Compiler Compilation ，它们是 Plugin Webpack之间的桥梁。 Compiler Compilation 的含义如下。

+ Compiler 对象包含了 Webpack 环境的所有配置信息，包含 options loaders plugins等信息。这个对象在 Webpack 启动时被实例化，它是全局唯一的，可以简单地将
它理解为 Webpack 实例。
+ Compilation 对象包含了当前的模块资源、编译生成资源、变化的文件等。当 Webpack以开发模式运行时，每当检测到一个文件发生变化，便有一次新的 Compilation
创建 Compilation 对象也提供了很多事件回调供插件进行扩展。通过 Compilation
也能读取到 Compiler 对象。

Compiler Compilation 的区别在于： Compiler 代表了整个 Webpack 从启动到关闭的生命周期，而 Compilation 只代表一次新的编译。


Webpack 通过 Tapable ( https://github.com/webpack/tapable ）来组织这条复杂的生产线Webpack 在运行的过程中会广播事件，插件只需要监听它所关心的事件，就能加入这条生产线中，去改变生产线的运作。 Webpack 的事件流机制保证了插件的有序性，使得整个系统的扩展性良好。

Webpack 的事件流机制应用了观察者模式，和 Node扣中的 EventEmitter 非常相似 Compiler Compilation 都继承自 Tapable ，可以直接在 Compiler Compilation 对象上广播和监昕事件，方法如下.


广播事件：

compiler.apply('event-name', params)

监听模式：

compiler.plugin('event-name', function(params))



