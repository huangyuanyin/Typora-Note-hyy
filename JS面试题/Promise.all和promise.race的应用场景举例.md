### 一、Promise.all 方法 

> ==Promise.all().then()== 适用于处理多个异步任务，且所有任务都能得到结果时的情况。

> 应用场景：用户点击按钮，会弹出一个弹出对话框，对话框中有两部分数据呈现，这两部分数据分别是不同的后端接口获取的数据。
>
> 弹框弹出后的初始情况下，就让这个弹出框处于==数据加载中==的状态，当这两部分数据都从接口获取到的时候，才让这个==数据加载中==状态消失。让用户看到这两部分的数据。

>那么此时，我们就需求这两个异步接口请求任务都完成的时候做处理，所以此时，使用Promise.all方法，就可以轻松的实现，我们来看一下代码写法：

```JavaScript
<template>
  <div class="box">
    <el-button type="primary" plain @click="clickFn">点开弹出框</el-button>
  </div>
</template>

<script>
	name:"App",
    methods:{
        clickFn(){
            this.alertMask = true; // 打开弹出框
      		this.loading = true; // 暂时还没数据，所以就呈现loading加载中效果
            
            // 第一个异步任务
            function asyncOne(){
                let async1 = new Promise(async (resolve,reject) => {
                    setTimeout(() => {
                        // 这里我们用定时器模拟后端发请求的返回的结果，毕竟都是异步的
						let apiData1 = "第一个接口返回数据啦!!!"
                        resolve(apiData1)
                    },800)
                })
                return async1
            }
             console.log("异步任务一", asyncOne());  // 返回的是一个Promise对象
            
            // 第二个异步任务
            function asyncTwo(){
                let async2 = new Promise(async (resolve,reject) => {
                    setTimeout(() => {
                        // 这里我们用定时器模拟后端发请求的返回的结果，毕竟都是异步的
						let apiData2 = "第二个接口返回数据啦!!!"
                        resolve(apiData2)
                    },700)
                })
                return async2
            }
            console.log("异步任务二", asyncTwo()); // 返回的是一个Promise对象
            
            let paramsArr = [asyncOne(),asyncTwo()]
            
            // Promise.all方法接收的参数是一个数组，数组中的每一项是一个个的Promise对象
      		// 我们在 .then方法里面可以取到 .all的结果。这个结果是一个数组，数组中的每一项
      		// 对应的就是 .all数组中的每一项的请求结果返回的值
            Promise
            	.all(paramsArr)
            	.then((value) => {
                	console.log("Promise.all方法的结果", value);
                	this.loading = true // 现在有数据了，所以就关闭loading加载中效果
            	})
            
        }
    }
</script>
```

>  打印的结果图：

![结果](C:\Users\huangyuanyin\Pictures\结果.jpg)



### 二、Promise.race 方法

> Promise.race赛跑机制，只认第一名

> 应用场景：**点击按钮发请求，当后端的接口超过一定时间，假设超过三秒，没有返回结果，我们就提示用户请求超时**

```JavaScript
<template>
  <div class="box">
    <el-button type="primary" plain @click="clickFn">点击测试</el-button>
  </div>
</template>

<script>
export default {
  name: "App",
  methods: {
    async clickFn() {
      // 第一个异步任务
      function asyncOne() {
        let async1 = new Promise(async (resolve, reject) => {
          setTimeout(() => {
            // 这里我们用定时器模拟后端发请求的返回的结果，毕竟都是异步的
            let apiData1 = "某个请求";
            resolve(apiData1);
          }, 4000);
        });
        return async1;
      }
      console.log("异步任务一", asyncOne());  // 返回的是pending状态的Promise对象

      // 第二个异步任务
      function asyncTwo() {
        let async2 = new Promise(async (resolve, reject) => {
          setTimeout(() => {
            let apiData2 = "超时提示";
            resolve(apiData2);
          }, 3000);
        });
        return async2;
      }
      console.log("异步任务二", asyncTwo()); // 返回的是pending状态的Promise对象

      // Promise.race接收的参数也是数组，和Promise.all类似。只不过race方法得到的结果只有一个
      // 就是谁跑的快，结果就使用谁的值
      let paramsArr = [asyncOne(), asyncTwo()]

      Promise
      .race(paramsArr)
      .then((value) => {
        console.log("Promise.race方法的结果", value);
        if (value == "超时提示") {
          this.$message({
            type:"warning",
            message:"接口请求超时了"
          })  
        }else{
          console.log('正常操作即可');
        }
      })
    },
  },
};
</script>

```

### 总结：

- Promise.all接收的是数组，得到的结果也是数组，并且一一对应，也可以理解为Promise.all照顾跑的最慢的，最慢的跑完才结束。
- Promise.race接收的也是数组，不过，得到的却是数组中跑的最快的那个，当最快的一跑完就立马结束。



























