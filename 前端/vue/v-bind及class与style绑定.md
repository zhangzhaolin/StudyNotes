[TOC]

# v-bind及class与style绑定

## 绑定class的几种方式

```php+HTML
<body>
	<div id="app">
		<button v-bind:class="{'active':isActive}" :style="{'display':isActive}">
			active_class_bind
		</button>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : document.getElementById('app'),
		data : {
			isActive : true
		}
	});
</script>
```

类名active依赖于数据`isActive`，当其为true时，div会拥有类名active，当其为false时候则没有，下面是在谷歌浏览器中的验证 ：

```html
> app.isActive = false
<button class="">
	active_class_bind
</button
```

对象中也可以传入多个属性，`:class`与普通`class`可以共存：

```html
<button class="static" v-bind:class="{'active':isActive,'error':isError}">
	something
</button>
...
<script type="text/javascript">
	var app = new Vue({
		el : document.getElementById('app'),
		data : {
			isActive : true,
			isError : false,
		}
	});
</script>
```

当`:class`的每一项都为true时，对应的类名就会加载，以上的渲染结果为：

```html
<button class="static active">
	something
</button>
```

⚠ **当`:class`的表达式过长或者逻辑较为复杂的时候，可以绑定一个计算属性**

```php+HTML
<body>
	<div id="app">
		<div :class="classes">
			你们最近有些戏剧
		</div>	
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		el : document.getElementById("app"),
		data : {
			isActive : true,
			error : null
		},
		computed : {
			classes : function(){
				return {
					active : this.isActive && !this.error,
					'text-fail':this.error && (this.error.type === 'fail')
				}
			}
		}
	})
</script>
```

网页渲染为 ：

```html
<div class="active">
    你们最近有些戏剧
</div>
```

如果更改为如下格式的话 ：

```javascript
var app = new Vue({
		el : document.getElementById("app"),
		data : {
			isActive : true,
			error : {
				type : "fail"
			}
		},
		computed : {
			classes : function(){
				return {
					active : this.isActive && !this.error,
					'text-fail':this.error && (this.error.type === 'fail')
				}
			}
		}
	})
```

那么渲染会变为 ：

```html
<div class="text-fail">
	你们最近有些戏剧
</div>
```

## 数组语法

当需要多个class的时候，可以使用数组语法，给`:class`绑定一个数组：

```html
<body>
	<div id="app">
		
		<div :class="[activeCls,errorCls]">
		</div>

	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		// el : docuemnt.getElementById("#app"),
		el : "#app",
		data : {
			"activeCls" : "active",
			"errorCls" : "error"
		}
	})
</script>
```

还可以在数组中加入表达式 ：

```html
<body>
	<div id="app">
		<div :class="[isActive?activeCls:'',errorCls]">
			对我的批评尽情开展
		</div>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		// el : docuemnt.getElementById("#app"),
		el : "#app",
		data : {
			isActive : false,
			activeCls : "active",
			errorCls : "error"
		}
	})
</script>
```

或者使用对象语法 ：

```html
<div :class="[{activeCls : isActive},errorCls]">
	我太多地方不如你们
</div>
```

当然，还可以使用计算属性或者methods：

```html
<body>
	<div id="app">
		<button :class="classes">
			something
		</button>
	</div>
</body>
<script type="text/javascript">
	var app = new Vue({
		// el : docuemnt.getElementById("#app"),
		el : "#app",
		data : {
			size : 'large',
			disabled : true
		},
		computed : {
			classes : function () {
				return [
					'btn',{
						['btn-'+this.size] : this.size !== '',
						['btn-disabled'] : this.disabled
					}
				];
			}		
		}
	})
</script>
```

示例中的样式`btn`始终会渲染，当数据`size`不为空的时候，会应用样式前缀`btn-`，并加上`size`的值；当数据`disabled`为`true`时，会应用样式`btn-disabled`

渲染结果为 ：

```html
<button class="btn btn-large btn-disabled">
    something
</button>
```

## 在组件上使用

TODO

## 绑定内联样式

使用`v-bind:style`（即`:style`）可以给元素绑定内联样式，方法与`:class`类似

```html
<body>
        <div id="app">
            <div :style="{'color':color,'font-size':fontSize + 'px'}">
                别刻意夸大缘分
            </div>
        </div>
    </body>
    <script type="text/javascript">
        var app = new Vue({
            el : "#app",
            data : {
                "color" : "red",
                "fontSize" : "31"
            }
        });
    </script>
```

💊 CSS属性名称使用驼峰命名或者短分隔符命名

大多情况下，直接写一大常串样式不利于阅读和维护，所以通常写到`data`或者`computed`中：

```html
<div :style="styles">
                面朝大海
            </div>
<script type="text/javascript">
        var app = new Vue({
            el : "#app",
            data : {
                styles : {
                    "color" : "red",
                    "font-size" : 31
                }
            }
        });
    </script>
```







