[TOC]

# 表单与v-model

## 基本用法

`v-model`用于表单类元素上双向绑定数据，会忽略所有表单元素的`value`、`checked`、`selected`特性的初始值，而总是将Vue实例的数据作为数据来源，所以应该在组件的`data`选项中声明初始值。

```html
<body>
    <div id="app" class="ui text container">
        <div class="ui raised segment">
            <h3> v-model </h3>
            <div class="ui input">
                <input type="text" v-model="message" value="something..." placeholder="输入..." />
            </div>
            <p>message : {{message}}</p>
        </div>

        <div class="ui raised segment">
            <h3>v-on:input || @input </h3>
            <div class="ui input">
                <!-- <input type="text" v-on:input="handleInput" placeholder="输入..." /> -->
                <input type="text" @input="handleInput" value="something..." placeholder="输入..."/>
            </div>
            <p>message : {{input_message}}</p>
        </div>

    </div>
</body>
<script type="text/javascript" src="../vue/vue.js"></script>
<script type="text/javascript" src="../semantic/jquery.js"></script>
<script type="text/javascript" src="../semantic/semantic.js"></script>
<script>
    var app = new Vue({
        el: "#app",
        data: {
            message: "",
            input_message: ""
        },
        methods : {
            handleInput : function(e){
                this.input_message = e.target.value; 
            }
        }
    })
</script>
```

`v-model`不会在中文输入法的拼音阶段更新数据，如果希望总是更新，可以使用`v-on:input`来代替`v-model`，例如：

```html
<div class="ui raised segment">
            <h3>v-on:input || @input </h3>
            <div class="ui input">
                <!-- <input type="text" v-on:input="handleInput" placeholder="输入..." /> -->
                <input type="text" @input="handleInput" value="something..." placeholder="输入..."/>
            </div>
            <p>message : {{input_message}}</p>
        </div>
```

## 单选按钮

```html
<div id="app" class="ui raised segment">
            <h3>单选按钮</h3>
            <div class="ui radio checkbox">
                <input type="radio" name="radio" :checked="picked">
                <label for="radio">单选按钮</label>
            </div>
        </div>
<script>
    var app = new Vue({
        el: "#app",
        data: {
            picked: false
        }
    });
</script>
```

## 单选按钮组合使用

如果是单选按钮组合使用来表示互斥效果，那就需要使用`v-model`了：

```html
<div id="app1" class="ui raised segment">
    <h3>单选框</h3>
	<input type="radio" v-model="picked" value="html" />
    <label for="html">html</label>
    
    <input type="radio" v-model="picked" value="js" />
    <label for="js">javascript</label>
    
    <input type="radio" v-model="picked" value="css" />
    <label for="css">css</label>
	<label>选择的项是 : {{picked}} </label>
</div>
<script>
	var app1 = new Vue({
        el: "#app1",
        data: {
            picked: "html"
        }
    });
</script>
```

## 复选框组合使用

```html
<div id="app4" class="ui raised segment">
	<h3>复选框组合</h3>
	<input type="checkbox" v-model="picked" value="html" />
    <label for="html">html</label>
    
    <input type="checkbox" v-model="picked" value="js" />
    <label for="js">javascript</label>
                      
    <input type="checkbox" v-model="picked" value="css" />
    <label for="css">css</label>
                       
    <label>选择的项是 : {{picked}} </label>
</div>
<script>
	var app4 = new Vue({
        el: "#app4",
        data: {
            picked: ["html", "css"]
        }
    });
</script>
```

## 下拉列表

```html
<div id="app5" class="ui raised segment">
	<h3>下拉框列表</h3>
	<select v-model="picked" class="ui fluid dropdown" multiple>
        <option></option>
        <option value="angular">Angular</option>
        <option value="css">CSS</option>
		<option value="html">HTML</option>
    </select>
	<lable>选中的是 : {{picked}}</lable>
</div>
<script>
	var app5 = new Vue({
        el: "#app5",
        data: {
            picked: ["angular","html"]
        }
    });
</script>
```

**🐲 其中`multiple`可以设置多选。**

效果如下图所示 ：

![效果图](C:\Users\zhang\Desktop\TIM截图20180917152425.png)

## 绑定值



```html
<!-- 当选中时，`picked` 为字符串 "a" -->
<input type="radio" v-model="picked" value="a">
```

例如 ：

```html
<body>
    <div class="ui text container" style="padding-top:10px;">
        <div id="app" class="ui raised segment">
            <h3>单选按钮</h3>
            <div class="ui radio checkbox">
                <input type="radio" name="radio" v-model="picked" :value="value"/>
                <label for="radio">单选按钮</label>
            </div>
            <p> {{picked}} </p>
            <p> {{value}} </p>
        </div>
    </div>
</body>
<script type="text/javascript" src="../vue/vue.js"></script>
<script type="text/javascript" src="../semantic/jquery.js"></script>
<script type="text/javascript" src="../semantic/semantic.js"></script>
<script>
    var app = new Vue({
        el : "#app",
        data : {
            picked : false,
            value : "123"
        }
    })
</script>
```

当单选按钮被选中的时候，`picked`的值会变为`123`

如果是复选框的话 ：

```html
<!-- 当选中时候为yes 没有选中时候为no -->
<input type="checkbox" v-model="toggle" true-value="yes" false-value="no"/>
```

例如 ：

```html
<div id="app1" class="ui raised segment">
            <h3>复选框</h3>
            <div class="ui checkbox">
                <input type="checkbox" v-model="picked" :true-value="value1" :false-value="value2"/>
                <label for="radio">复选框</label>
            </div>
            <p> {{picked}} </p>
</div>
<script>
	var app1 = new Vue({
        el : "#app1",
        data : {
            picked : false,
            value1 : 'a',
            value2 : 'b'
        }
    });
</script>
```

勾选的时候，值为a，取消勾选的时候，值为b。

```html
<select v-model="selected">
    <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```

当选中下拉框选项的时候

```javascript
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

## 修饰符



