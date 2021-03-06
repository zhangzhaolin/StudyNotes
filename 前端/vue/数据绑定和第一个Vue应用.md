[TOC]

# 数据绑定和第一个Vue应用

## 数据绑定

```html
<!DOCTYPE html>
<html>
<head>
	<title>yourname</title>
	<script type="text/javascript" src="../vue/vue.js"></script>
</head>
<body>
	<div id="app" class="div">
		<input type="text" name="" placeholder="yourname" v-model="name">
		<h2>hello! {{name}}</h2>	
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			name : "苟利国家生死以"
		}
	})
</script>
</html>
```

选项`el`用于指定一个页面已存在的`DOM`元素来挂载`VUE`实例，它可以是一个`HTMLElement`，也可以是一个选择器，例如 ，以下三种情况都可以：

```javascript
var app = new Vue({
	// el : "#app",
	// el : document.getElementById('app'),
	// el : document.getElementsByClassName('div')[0],
});
```

**✅挂载成功后，可以通过`this.$el`访问该元素**：

```javas
var app = new Vue({
    el : document.getElementById("app"),
    data : {
        name : "苟利国家生死以..."
    },
    mounted : function(){
        console.log(this.$el);
    }
})
```

> 除了数据属性，`Vue` 实例还暴露了一些有用的实例属性与方法。它们都有前缀 `$`，以便与用户定义的属性区分开来。

`VUE`数据绑定默认为 **双向数据绑定**：

```javascript
var myData = {
    a : 1
};
var app = new Vue({
    el : "#app",
    data : myData
});
console.log(app.myData); // 1
app.a = 7;
console.log(myData.a);	// 7
myData.a = 3;
console.log(app.a);		// 3
```

但是例外是 —— **使用[`Object.freeze`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)会阻止修改现有的属性** :

```html
<body>
    <div id="app">
        <p>{{foo}}</p>
        <button @click="foo = 'baz'"> 修改 </button>
    </div>
</body>
<script src="../vue/vue.js"></script>
<script>
    var obj = {
        foo: "bar"
    };
    Object.freeze(obj);
    var app = new Vue({
        el: "#app",
        data: obj
    });
</script>
```

## 生命周期

`Vue`生命周期中比较常用的有 ：

1. `created` ：实例完成之后调用，此阶段完成了数据的观测，但尚未挂载，`$el`还不可用。
2. `mounted` ： `el`挂载到实例上后调用。
3. `beforeDestory` ：实例销毁前调用，主要解绑一些使用`addElementListener`监听的事件等。

```html
<!DOCTYPE html>
<html>
<head>
	<title>lifecycle</title>
	<script type="text/javascript" src="../vue/vue.js"></script>
</head>
<body>
	<div id="app"></div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			a : 3
		},
		created : function() {
			console.log(this.a);   // 3
			console.log(this.$el); // undefined
		},
		mounted : function(){
			console.log(this.$el); // <div id="app"></div>
		},
		beforeDestory : function() {
			// something...
		}
	})
</script>
</html>
```

## 插值与表达式

⏰ 实时显示当前时间 ：

```html
<!DOCTYPE html>
<html>
<head>
	<title>time</title>
	<script type="text/javascript" src="../vue/vue.js"></script>
</head>
<body>
	<div id="app">
		{{date}}
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			date : new Date()
		},
		mounted : function() {
			var _this = this;
			_this.timer = setInterval(function(){
				_this.date = new Date();
			},1000);
		},
		beforeDestory : function() {
			if(this.timer){
				clearInterval(this.timer);	
			}
		}
	})
</script>
</html>
```

如果你想输出一段HTML，而不是纯文本 。可以使用`v-html` ：

```html
<!DOCTYPE html>
<html>
<head>
	<title>printHtml</title>
	<script type="text/javascript" src="../vue/vue.js"></script>
</head>
<body>
	<div id="app">
		<p v-html="link"></p>
	</div>
</body>
<script type="text/javascript">
	new Vue({
		el : "#app",
		data : {
			link : "<a href='#'>this is a link</a>"
		}
	})
</script>
</html>
```

如果想显示`{{..}}`标签，而不想进行替换，可以使用`v-pre`：

```html
<!DOCTYPE html>
<html>
<head>
	<title>uncompile</title>
	<script type="text/javascript" src="../vue/vue.js"></script>
</head>
<body>
	<div id="app">
		<p>{{words}}</p>
		<p v-pre>{{this is a uncomplie words.}}</p>
	</div>
</body>
<script type="text/javascript">
	new Vue({
		el : "#app",
		data : {
			words : "this is a compile words."
		}
	})
</script>
</html>
```

`{{...}}`中，除了简单的绑定属性值之外，还可以使用JavaScript表达式进行简单的运算 ：

```html
<body>
	<div id="app">
		<p> {{number/10}} </p> 
        <!-- 100.1 -->
		<p> {{isOk? "确定" :"取消"}} </p>
		<!-- 取消 -->
		<p> {{text.split(",").reverse().join(",")}}  </p>
        <!-- 456,123 -->
	</div>
</body>
<script type="text/javascript">
	new Vue({
		el : "#app",
		data : {
			number : 1001,
			isOk : false,
			text : "123,456"
		}
	})
</script>
```

🚨`Vue`**只支持单个表达式，不支持语句和流程控制**

## 过滤器

```html
<!DOCTYPE html>
<html>
<head>
    <title>datefilter</title>
    <script type="text/javascript" src="../vue/vue.js"></script>
    <script type="text/javascript" src="../vue/dateFormat.js"></script>
</head>
<body>
    <div id="app">
        {{date | formatDate}}
    </div>
</body>
<script type="text/javascript">
    var app = new Vue({
        el : document.getElementById('app'),
        data : {
            date : new Date()
        },
        filters : {
            formatDate : function(value) {
                return DateFormat.format.date(value , "yyyy/MM/dd HH:mm:ss");
            }
        },
        mounted : function () {
            var _this = this;
            _this.timer = setInterval(function() {
                _this.date = new Date();
            },1000);
        },
        beforeDestory : function () {
            if(this.timer){
                clearInterval(this.timer);
            }
        }
    })
</script>
</html>
```

PS ：[**jquery-dateFormat**](https://github.com/phstc/jquery-dateFormat#examples)

```javascript
{{ date | formatDate }}

var app = new Vue({
    el : "#app",
    data : {
        date : new Date()
    },
    filters : {
        formatDate : function(value){
            return something;
        }
    }
})
```

💡 **过滤器也可以串联，并且可以接收参数**

```html
<!-- 串联 -->
{{message | filterA | filterB}}
<!-- 接收参数 -->
{{message | filter('arg1','arg2')}}
```

这里的字符串`arg1`和`arg2`将分别传给过滤器的第二个和第三个参数，因为第一个是数据本身。

## 指令与事件

🤞 **指令带有前缀`v-`**

```php+HTML
<body>
	<div id="app">
		<p v-if="show">show this words.</p>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			show : true
		}
	})
</script>
```

`v-bind`的基本用途是 ：动态更新HTML元素上的属性，比如id、class等 ：

```html
<body>
	<div id="app">
		<img v-bind:src="imgUrl"/>
		<p><a v-bind:href="githubUrl">大型交友网站</a></p>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			imgUrl : "http://www.uml.org.cn/safe/images/2018022724.jpg",
			githubUrl : "https://github.com"
		}
	})
</script>
```

`v-on` ：用来绑定事件监听器 

```html
<body>
	<div id="app">
		<!-- <p> {{show?"这是一段文本":""}} </p> -->
		<p v-if="show">这是一段文本.</p>
		<button v-on:click="handleClose">点击隐藏</button>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			show : true
		},
		methods : {
			handleClose : function() {
				this.show = false;
			}
		}
		
	})
</script>
```

或者可以这样编写 ：

```html
<body>
	<div id="app">
		<!-- <p> {{show?"这是一段文本":""}} </p> -->
		<p v-if="show">这是一段文本.</p>
		<!-- <button v-on:click="handleClose">点击隐藏</button> -->
		<button v-on:click="show = false">点击隐藏</button>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : "#app",
		data : {
			show : true
		},
		methods : {
			handleClose : function() {
				this.show = false;
			}
		}
		
	})
</script>
```

在`methods`中也可以调用方法 ：

```javascript
var app = new Vue({
		el : "#app",
		data : {
			show : true
		},
		methods : {
			handleClose : function() {
				this.close();
			},
			close : function(){
				this.show = false;
			}
		}
		
	});
```

## 🍨 语法糖

`vue`中的`v-bind`以及`v-on`提供了语法糖 ：

```html
<!-- <img v-bind:src="imgUrl"/>
<p><a v-bind:href="githubUrl">大型交友网站</a></p> -->
<img :src="imgUrl">
<p><a :href="githubUrl">大型交友网站</a></p>
```

```html
<!-- <button v-on:click="handleClose">点击隐藏</button> -->
<button @click="handleClose">点击隐藏</button>
```



