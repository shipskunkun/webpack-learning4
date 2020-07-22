## 4章，优化

(1)优化开发体验

+ 优化构建速度，
+ 优化使用体验，通过自动化手段完成一些重复的工作，让我们专注于解决问题本身

(2)优化输出质量	
优化输出质量的目的是为用户呈现体验更好的网页，例如减少首屏加载时间、提升性能
流畅度等

+ 减少用户能感知到的加载时间，也就是首屏加载时间
+ 提升流畅度，也就是提升代码性能。


### 4.1 缩小文件搜索范围

4.1.1 优化 Loader 配置

思考一下，loader 的参数有哪些？

```
module.exports = {
	entry: {},
	mode: 
	module: {
		rules: [{
			test: /\.js$/,
			use: {
				loader: ['babel-loader? cacheDirectory']
				options:
			}
			include: path.resolve(__dirname, 'src')
		}]
	}

}
```

+ test： 如果项目源码中只有js文件，就不要写成／＼.jsx$／ ，以提升正则表达式的性能
+ loader: 支持缓存转换出的结果，通过 cacheDirectory 选项开启
+ include: loader 选择范围，只对 src 下的 js 文件使用 babel-loader



4.1.2 优化 resolve modules 配置

resolve.modules 中，配置webpack去哪些目录下寻找第三方模块

resolve.modules 的默认值是`［ node_modules ］`，含义是先去当前目录的 `node_modules` 目录下去找我们想找的模块，如果没找到 就去上一级 目录 `./node_modules`
中找，再没有就去 `../../node_modules` 中找，以此类推 这和 node.js 的模块寻找机制很相似。

使用绝对路径指明第三方模块存放的位置，以减少搜索步骤



```
module.exports = { 
resolve : {
		//使用绝对路径指明第三方模块存放的位置，以减少搜索步骤
		//其中， dirname 表示当前工作目录，也就是项目根目录
		modules: [path.resolve(__dirname, 'node modules')]
		}
};
```



4.1.4 优化 resolve.alias 配置

4.1.5 优化 resolve.extensions 配置

在导入语句没带文件后缀时， webpack 会在自动带上后缀后去尝试询问文件是否存在。
2.4 中介绍过， resolve.extensions 用于配置在尝试过程中用到的后缀列表，默认
是：

```
extensions :[’.js’,’.json’]
```
在配置 resolve.extensions时需要遵守 以下几点，以做到尽可能地优化构建性能。

+ 后缀尝试列表要尽可能小，不要将项目中不可能存在的情况写到后缀尝试列表中。
+ 频率出现最高的文件后缀要优先放在最前面，以做到尽快退出寻找过程。
+ 在源码中写导入语句时，要尽可能带上后缀 从而可以避免寻找过程。例如在确定
的情况下将 require ( ’. /data ’） 写成 require （’. data.json ’）

```
modu le.exports = { 
resolve : { 
	／／尽可能减少后缀尝试的可能性
	extensions : [’js’],
	}
}
```



4.1.6 module.noParse

module.noParse 配置项可 Webpack 忽略对部分没采用模块化的文件的递归解析处理，这样做的好处是能提高构建性能。比如 react.min.js



```js
module.exports ={
​	module: {
​		noParse: [/react\.min\.js$],
​	}
}
```



### 4.2  使用DllPlugin

包含大量复用模块的动态链接库只需被编译一次，在之后的构建过程中被动态链接库包含的

模块将不会重新编译，而是直接使用动态链接库中 的代码 由于动态链接库中大多数包含的

是常用的第 方模块，例如 react、 react-dom ，所以只要不升级这些模块的版本，动态链接

库就不用重新编译。



### 4.3 happyPack

它将任务分解给多个子进程去并发执行，子进程处理完后再将结果发送给主进程。

由于 JavaScript 是单线程模型，所以要想发挥多核 CPU 的功能，就只能通过多进程实现，而无法通过多线程实现。



+ 在loader配置中，对所有文件处理都交给了happy/loader， 使用紧跟其后的 querystring ? id=babel 去告诉 happypack/loader 选择哪个 HappPack 实例处理文件。
+ 在plugins 中，新增了两个 HappyPack 实例，分别用于告诉 happypack/loader 如何处理 .js 和 .css 文件。选项中的id属性的值和上面的querystring 中的 id = babel对应，loaders和Loader 配置中的一样



```
module.exports = { 
	module : { 
    rules: [ 
    	{
        test: /\.js$／， 
        ／／将对.js 文件的处理转交给 id babel HappyPack 实例
        use: ['happypack/loader?id=babel'], 
        ／／ 排除 node modules 目录下的文件 node modules 目录下的文件都采用了 ESS
        语法，没必要再通过 Babel 去转换
        exclude: path.resolve( __dirname,'node modules'),
      },
      {
        ／／将对 .css 文件的处理转交给 id css HappyPack 实例
        test: /\.css$/ , 
        use: ExtractTextPlugin.extract({ 
        	use: [’ happypack/loader?id=css ’],
       }),
      },
     ]
  }, 
  plugins: [ 
    new HappyPack ( { 
      ／／用唯 的标识符 id 来代表当前的 HappyPack 用来处理 类特定的文件
      id:’babel ’, 
      ／／如何处理. j 文件，用法和 Loader 配置中一样
      loaders: [’ babel-loader?cacheDirectory ’], 
      ／／使用共享进程池中的子进程去处理任务
      threadPool : happyThreadPool, 
    } ), 
    new HappyPack({ 
      id:’css ’ , 
      ／／如何处理 css 文件，用法和 Loader 配置中的一样
      loaders: [ ’ css-loader ’], 
      ／／使用共享进程池中的子进程去处理任务
      threadPool : happyThreadPool, 
    }), 
    new ExtractTextPlugin({ 
    	filename: `[name].css`,
    }),
   ],
 }
```



### 4.4 使用 ParallelUglifyPlugin

并行处理的思想也引入到代码压缩中



当Webpack 有多个 JavaScript 文件需要输出和压缩时，原本会使用 UglifyJS 去一个一个压缩再输出，但是 ParallelUglify Plugin 会开启多个子进程，将对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 UglifyJS 去压缩代码，但是变成了并行执行。所ParallelUglify Plugin 能更快地完成对多个文件的压缩工作。



### 4.5 使用自动刷新 

webpack中监昕 个文件发生变化的原理，是定时获取这个文件的最后编辑时间，每次都存下最新的最后编辑时间，如果发现当前获取的和最后一次保存的最后编辑时间不一致，就认为该文件发生了变化。配置项中的 watchOptions.poll 用于控制定时检查的周期，具体含义是每秒检查多少次。



当发现某个文件发生了变 时，并不会立刻告诉监昕者，而是先缓存起来，收集一段时间的变化后，再一次性告诉监听者。配置项中的 watchOptions.aggregateTime out于配置这个等待时间。



### 4.6 开启模块热替换

不刷新整个网页的情况下做到超灵敏实时预览。原理是在一个源码发生变化时，只需重新编译发生变化的模块，再用新输的模块替换掉浏览器中对应的老模块

### 4.7 区分环境

if (process.env.NODE ENV === ’ production ’){ 

​	console.log （’你正在线上环境 ）；

) }

else { 

​	console . log （’你正在使用开发环境 ）；

}

### 4.8 压缩代码

4.8.1 压缩 JavaScript

UglifyJsPlugin ：通过封装 UglifyJS 实现压缩

ParallelU glify Plugin 多进程并行处理压缩，在 4.4 节中有详细介绍

### 4.9 CDN 加速

CDN （内容分发网络）的作用就是加速网络传输，通过将资源部署到世界各地，使用户在访问时按照就近原则

从离其最近的服务器获取资源，来加快资源的获取速度。



1. 针对 HTML 文件：不开启缓存，将 HTML 放到自己的服务器上，而不是 CDN 服务上，同时关闭自己服务器上的缓存。自己的服务器只提供 HTML 文件和数据接口。

2. 针对静态的 JavaScript 、css 、图片等文件 开启 DN 缓存，上传到 CDN上，同时为每个文件名带上由文件内容算出的 hash 值，例如上面的 app a6976b6d . css 文件。带上 Hash 值的原因是文件名会随着文件的内容而变化，只要文件的内容发生变化，其对应的 也就会变化，它就会被重新下载，无论缓存时间有多长



除此之外，如果我们还知道浏览器有一个规则是，在同一 时刻针对同 一个域名的资源的并行请求有限制（大概 4个左右，不同的浏览器可能不同），则会发现上面的做法有很大的问题。

由于所有静态资源都被放到了同一个 CDN 服务的域名下，也就是上面的 cdn.com下，如果网页的资源很多，例如有很多图片，就会导致资源的加载被阻塞，因为同时只能加载几个， 必须等其他资源加载完才能继续加载。

要解决这个问题，我们可以将这些静态资源分散到不 同的 CDN 服务上，例如将 JavaScript 文件放到 js.cdn.com 域名下，将 css 文件放到 css.cdn.com 域名下，将图片文件放到 img.cdn.com 域名下，这样 index .html

使用多个域名后又会带来一个新的问题：增加域名解析时间。对于是否采用多域名分散资源，需要根据自己的需求去衡量得失。当然 ，可以通过在 HTML HEAD 标签中加入＜ link rel="dns-prefetch href="js.cdn.com"＞预解析域名，以减少域名解析带来的延迟。



#### 4.9.3 Web pack 实现 CON 的接入

总之，构建需要实现以下几点。

+ 静态资源的导入URL 需要变成指向 CDN 服务的绝对路径的 URL，而不是相对HTML 文件的 URL

+ 静态资源的文件名需要带上由文件内容算出来的 Hash 值，以防止被缓存

+ 将不同类型的资源放到不同域名的 DN 服务上，以防止资源的并行加载被阻塞



js 一个 publicPath

css 文件一个 publicPath

html 文件 一个 stylePublicPath



在以上代码中最核心的部分是通过 publicPath 参数设置存放静态资源的 CDN 目录URL ,为了让不同类型的资源输出到不同的 CON ，需要分别进行如下设置

+  output publicPath 中设置 JavaScript 的地址

+ css-loader.publicPath 中设置被 css 导入的资源的地址。

+ WebPlugin.stylePublicPath 中设置 css 文件的地址。



### 4.10 使用 Tree Shaking 

### 4.11 提取公共代码

1. 提取公共代码

   ​	slitChunks

2. 按需加载

   ​	在为单页应用做按需加载优化时， 般采用以下原则。

   + 将整个网站划分成一个个小功能，再按照每个功能的相关程度将它们分成几类

   + 将每一类合并为一个Chunk ，按需加载对应的 chunck

   + 不要按需加载用户首次打开网站时需要看到的画面所对应的功能，将其放到执行入

   口所在的 Chunk 中，以减少用户能感知的网页加载时间。

   + 对于不依赖大量代码的功能点，例如依赖 Chart.js 去画图表、依赖 flv js 去播放视

   频的功能点，可再对其进行按需加载。

   

被分割出去的代码的加载需要 一定的时机去触发，即当用户操作到了或者即将操作到对应的功能时再去加载对应的代码。被分割出去的代码的加载时机需要开发者根据网页的需求去衡量和确定。

由于被分割出去进行按需加载的代码在加载的过程中也需要耗时，所以可以预估用户接下来可能会进行的操作，并提前加载对应的代码，让用户感知不到网络加载.





webpack 内置了强大的分割代码的功能去实现按需加载，实现起来非常简单 。举个例子，

现在需要做这样一个进行了按需加载优化的网页。

+ 网页首次加载时只加载 main.js 文件，网页会展示一个按钮，在 main.j 文件中只包含监昕按钮事件和加载按需加载的代码。

+ 在按钮被单击时才去加载被分割出去的 show.js文件，在加载成功后再执show . js 里的函数。

  

```
window.document.getElementByid('btn').addEventListener ('click', 

  function (){ 

  	//在按钮被单击后才去加载 show.js 文件，文件加载成功后执行文件导出的函数

 	 import(/* webpackChunkName :”show " */'./show').then ((show) => { 

  		show ('Webpack); 
  }) 
}) ; 

show.js 文件的内容如下：

module.exports = function (content) { 
	window.alert (’ Hell 0 ’ + content); 
};
其中最关键的一句是：

import(/* webpackChunkName : ” show " */ './show'>

```

Webpack 内置了对 import(*)语句的支持，当 Wepack 遇到了类似的语句时会这样

+ 以 ./ show.js 为入口重新生成一个 Chunk
+ 代码执行到 import 所在的语句时才去加载由 Chunk 对应生成的文件
+  import 返回一个 Promise ，当文件加载成功时可以在 Promise的 then 方法中获取 show.js 导出的内容。



需要配置 Webpack:

```
module.exports = { 
	//JavaScript 执行入口文件
  entry : { 
  	main :'./index.js' ,
  },
  output: { 
  	／／为从 entry 中配直生成的 Chunk 配置输出文件的名称
  	filename :’[name].js’, 
  	／／为动态加载 Chunk 配置输出文件的名称
  	chunkFilename : ’[name).js’,
   }
 };
```





### 4.14 scope hosting

可以看出开启 Scope Hoisting 后，函数申明由两个变成了一个， util. js 中定义 的内

容被直接注入 main.js对应的模块中。这样做的好处是：

+ 代码体积更小，因为函数申明语句会产生大量的代码：
+ 代码在运行时因为创建的函数作用域变少了 ，所以内 存开销也变小了。



Scope Hoisting 实现原理其实很简单：分析模块之间的依赖关系，尽可能将被打散的模块合并到一个函数中，但前提是不能造成代码元余. 因此只有那些被引用了 次的模块能被合井。



### 4.15 输出分析



webpack --profile --json > stats.json 