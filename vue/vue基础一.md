## 一、创建启动项目

1、通过脚手架创建

`npm create hellovue`

2、运行

`npm run serve`



3、开发先知：

```
在 src/views 下添加新的页面
在 src/router/index.js 下配置 import 即可

入口文件 main.js

单文件组件形式，在.vue文件中，<template>写h5,<script>写js，<style>写样式
```







## 二、基础语法

注意：列表用 [ ] ，对象用 { }

### 1、v-for

```vue
<p> {{"This is vue ," + msg }}</p>
<ul>
      <li v-for="item in list" :key="item"> {{ item }} </li>
</ul>
<p>
    {{ obj.name }}
</p>

<!--在data中-->
 data(){
    return {
      msgs:"你好",
      list:[1,2,3],
      obj:{
        id:1,
        name:"雷克斯"
      }
    }
  }
```



### 2、钩子和定时器

```vue
//渲染后就执行的钩子函数
    mounted(){
        //定时器，3s之后执行
        setTimeout(() => {
            this.msg = "hello leikes !" 
        },3000)
    }
```



### 3、绑定一个属性到标签中（很有用）v-bind | : 

```vue
<!-- 1、普通用法 -->
<div v-bind:id="id">
    这里的“id”不需要{{}},直接就是从data获取的！即：id=1
</div>

<script>
    data(){
        return{
            id:1
        }
    }
</script>

<!-- 2、可以简写!!! :属性名=值 -->
<div :id="id"> 
    
</div>

<!-- 3、动态修改绑定属性 -->
比如在data中
<script>
    data(){
        return{
            attrName:"id"
        }
    }
</script>
在div中
<div :[attrName]=1>
    此时绑定的属性就是 id=1，如果我们修改 attrName="msg"，那么此时就会自动变成 msg=1
</div>

```

### 4、绑定class

```vue
<div :class="classA">
    绑定一个class
</div>
<div :class="[classA ,classB]">
    绑定多个class
</div>

<div :class="objClass">
    通过对象绑定，只有为true的才会绑定进去
</div>

data(){
	return{
		classA:"class A",
		classB:"class B",
		objClass{
			current:true ,  //会绑定
			focus:false   ///不会绑定。
		}
	}
}

```



### 5、绑定style

```vue
<!-- 和绑定class差不多 -->
<div :style="{color:'red' , fontSize='36px'}">
   
</div>

<div :style="{color:color , fontSize='36px'}">
   	这个color就可以在data中动态改
</div>

一般也形成对象形式
<div :style="styObj">
    
</div>

data(){
return{
styObj{
	color："red",
	fontSize:26px
}
}
}
```



### 6、if \ else if \ else 

```vue
<!--下三者不显示时直接删除元素-->
v-if
v-else-if 
v-else


<!-- 不符合是通过 display:none -->
v-show

```



### 7、事件绑定

```vue
v-on:click="add"   ===   @click="add"  //可以不传
v-on:click="add(2)"  //可以传参

methods:{
	add(val){    // 这个val默认是传 event , 一个事件，所以我们要判断是否为数字才行。
		if(typeof val === 'number'){   //number，如果是数字
			this.num += val ;
		}else{
			this.num ++ ;
		}
	}
}



<!-- 如果还是想传递事件的话：$event -->
@click="add(2,$event)"

methods:{
	add(val,event){
		<!--这样就能在event读到传递过来的事件进行打印。-->
		console.log(event) ;
	}
}

newLists = [
	{id:1,name:'leike1'},{id:2,name:'leikes1'}
]
<!--如果要修改一个list中的整个对象的值，要这么修改：修改索引为0的-->
this.$set(this.newLists,0,{id:0,name:'leikes0'}) 
<!--动态修改对象属性也是如此-->
obj:{
     name:'leikes',
     age:20,
     addr:'广州'
}
// obj.hobby="吃饭睡觉打豆豆"  这种方式是无法新增属性的，需要：
this.$set(this.obj,'hobby','吃饭睡觉打豆豆')
<!--删除则是通过先拷贝一份，然后再删除，再重新赋值-->
let _obj = {...this.obj} ;
delete _obj.age;
this.obj = {..._obj}

```

**消除一些事件**

```vue
<!-- 阻止冒泡事件，子事件的触发会触发父事件的，如下,event2发生后会触发event1，通过 .stop阻止冒泡-->
<div @click="event1">
    <div @click.stop="event2">
        
    </div>
</div>

<!--也可以通过.self实现防止冒泡，只有点击自己才会触发事件，冒泡不会-->
<div @click.self="event1">
    <div @click="event2">
        
    </div>
</div>


<!--阻止事件，.prevent-->
<div @keydown.ctrl.prevent></div>  //阻止点击ctrl
<div @keydown.space.prevent></div> //阻止输入空格 同理，enter、right等




```



### 8、计算属性

```js

//computed的计算属性会缓存，调用一次就有结果，method的计算属性，每次调用method就会执行一次计算，性能差点。

<script>
    export default{
		name : "demo1" ,
         data(){
            return {
                msg : "demo1" ,
                type : "news" ,
                firstName : "张"
            }   
        },
            // 计算属性 computed 里面的都是计算好的，可以直接返回拼接后的结果
        computed:{
    		type_msg(){
        	return this.type + " : " + this.msg
    	}，
            //当然，也可以用方法进行计算属性
        method:{
            getFullName(){
                return this.firstName + "  : " + this.lastName 
            }
        }
	}
</script>



```





### 9、 数据双向绑定 v-model



### 10、过滤器 filter (可以定义全局过滤器)

```vue
<div>
    {{ content | myfilter }}
    {{ content | myfilter1（'args'） }}  <!-- 可以传参 -->
</div>


<script>
    export default{
        data(){
            return { 
            	content : "axsaxwxsx"
            }
        },
        filters:{
            myfilter(value){
                return value.toUpperCase()
            } ,
            myfilter1(value,type){
                return value.toUpperCase()
            }   
        }       
    }
</script>
    


```

















