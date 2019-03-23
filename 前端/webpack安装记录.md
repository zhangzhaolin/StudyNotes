# `webpack`安装记录

## 🎂安装`webpack`

1. 首先，创建一个目录，例如`npm-text`，进入此目录，执行命令：

   ```cmd
   npm init
   ```

   ![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/20190318/20190319140350.png)

   `package.json`内容如图所示：

   ![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/20190318/20190319140759.png)

2. 在本地局部安装`webpack`：

   ```cmd
   npm install webpack --save-dev
   ```

   安装成功后，`package.json`内容如图所示：
   ![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/20190318/20190319140945.png)

3. 接着安装`webpack-dev-server`，它可以在开发环境中提供很多服务，例如：启动一个服务器、热更新、接口代理等……，在本地局部安装：

   ```cmd
   npm install webpack-dev-server --save-dev
   ```

4. 安装`webpack-cli`

   ```cmd
   npm i -D webpack-cli
   ```

最终，`package.json`文件内容为：

```json
{
  "name": "npm-text",
  "version": "1.0.0",
  "description": "一个简单的npm测试",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack-dev-server --open --config webpack.config.js"
  },
  "author": "施瓦兹",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.29.6",
    "webpack-cli": "^3.3.0",
    "webpack-dev-server": "^3.2.1"
  }
}
```

## 🍕创建一个`js`文件

1. 在项目目录下创建一个名为`webpack.config.js`的文件：

   ```javascript
   var config = {
      
   }
   // export default config;
   module.exports = config;
   ```

2. 在`package.json`中的`scripts`中增加一个快速启动`webpack-dev-server`服务的脚本：

   ```javascipt
   {
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
   	"dev" : "webpack-dev-server --open --config webpack.config.js"
     },
   }
   ```

   当我们运行`npm run dev`命令时，就会执行`webpack-dev-server --open --config webpack.config.js`命令。其中`--config`是指向`webpack-dev-server`读取的配置文件路径，在这里指向了`webpack.config.js`。`open`会在执行命令时自动在浏览器打开页面，默认地址是`127.0.0.1:8080`，不过IP和端口号都是可以配置的，例如：

   ```javascript
   "dev": 
   "webpack-dev-server --host 192.168.1.101 --port 8888 --open --config webpack.config.js"
   ```

   `webpack`配置中最重要的两项是入口和出口。入口的作用是告诉`webpack`从哪里开始寻找依赖，并且编译。出口是用来配置编译后文件存储路径和文件名。

3. 在项目目录下新建一个空的`main.js`文件作为入口文件，然后再`webpack.config.js`中进行入口和输出的配置：

   ```javascript
   var path = require('path')
   var config = {
   	entry : {
           // home 是入口的名称
   		home: './main.js'
   	},
   	output : {
           // 输出目录 ：是一个相对目录
   		path : path.join(__dirname,'dist'),
            // 指定资源文件引用的目录
   		publicPath : '/dist/',
            // 指定每一个输出包的名称，如果有多个入口，可以使用入口的名称来为每一个包提供唯一名称
   		filename : '[name].js'
   	}
   }
   // export default config;
   module.exports = config;
   ```

4. 在项目目录下，新建一个`index.html`作为我们SPA（单页应用）入口：

   ```html
   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title></title>
   	</head>
   	<body>
   		Hello npm
   	</body>
   	<script type="text/javascript" src="/dist/home.js"></script>
   </html>
   ```

5. 在终端运行`npm run dev`命令：

   ![](https://raw.githubusercontent.com/zhangzhaolin/GraphBed/master/20190318/20190319160626.png)

现在，我们在`main.js`中加一行：

```javascript
document.getElementById("app").innerHTML = "Hello My Name Is ShiWaZi";
```

可以发现页面也随之发生了变化，这就是`webpack-dev-server`的热更新功能。

可以执行下面的命令对项目进行打包：

```cmd
webpack --progress --hide-modules
```

## 🎮`webpack.config.js`还可以更简洁

在做上面的例子时候，我通过官方网站，发现自己的`webpack.config.js`还可以更加简洁：

```javascript
const path = require('path')

module.exports = {
	entry : {
		home: './main.js'
	},
	output : {
		path : path.join(__dirname,'dist'),
		publicPath : '/dist/',
		filename : '[name].js'
	}
};
```

## 🏗完善配置文件

在`webpack`的世界中，每一个文件都是一个模块。例如：CSS、HTML、JPG、JS等。对于不同的模块，需要不同的加载器（`Loaders`）来处理。

例如，现在要写一些CSS样式，就要用到`style-loader`以及`css-loader`：

```
npm install css-loader --save-dev
npm install style-loader --save-dev
```

配置完成后，需要在`webpack.config.js`文件里进行相应的配置，增加对`.css`的处理：

```js
const path = require('path')

module.exports = {
	entry: {
		home: './main.js'
	},
	module: {
        // rules 属性可以指定一系列的loaders
        // 每一个loader都必须包含test和use两个选项 
		rules: [{
			test: /\.css$/,
            // 编译顺序是从后往前
			use: ['style-loader', 'css-loader'],
		}],
	}
	output: {
		path: path.join(__dirname, 'dist'),
		publicPath: '/dist/',
		filename: '[name].js'
	}
};
```

这段配置的意思是，当webpack编译过程中遇到`require()`或者`import`语句导入一个后缀名为`.css`的文件时，先将它通过`css-loader`转换，再通过`style-loader`转换，然后继续打包。

在项目目录下面新建一个`style.css`文件，并在`main.js`中导入：

```
/* style.css */
#app{
	font-size: 3rem;
	color: indianred;
	font-family: "微软雅黑";
}

/* main.js */
import './style.css';

document.getElementById("app").innerHTML = "Hello My Name Is ShiWaZi";
```

可以看到，CSS是通过`JavaScript`动态创建`<style>`标签来写入的，但在实际业务中，可能并不希望这样做。我们可以使用一个`webpack`插件来把散落的`css`提取出来，并形成一个`main.css`文件，插件名字为：`MiniCssExtractPlugin`

安装：

```npm
    npm install --save-dev mini-css-extract-plugin
```





