# 工作中遇到的一些问题和经验总结

1. IE会对get请求进行缓存，如果需要经常更新数据的get接口，需要在接口后面加上时间戳，可以统一在请求的拦截器上加

 ```
   // 请求拦截器
http.interceptors.request.use(function (config) {
  config.url += '?t=' + Date.now() // 兼容IE
  return config
}, function (error) {
  // 对请求错误做些什么
  return Promise.reject(error)
})
```

2. 假如用到第三方库，无法用在vue中，如果之前我们的地图库，我们可以在vue中用iframe嵌套jsp页面方法，第三方库在jsp中使用。

但两个工程端口不一样，根据浏览器的同源策略，iframe会产生dom跨域，最麻烦的调试方法就是Vue打包到jsp里查看效果，但这种调试方式大大影响效率，因为开发的时候肯定会有各种问题，需要多次调试。document.domain可以解决iframe跨域问题。

vue和jsp肯定需要互相交互，vue中调用jsp的方法可以通过iframe的调用父子级调到，jsp调用vue的方法就比较麻烦，这个可以用jquery自定义事件解决

```
// 自定义事件
$(document.body).on('alarm', () => {
  this.alarmEventList(1, 200, 0);
})
// 触发事件
$(document.body).trigger('alarm');
// 组件关闭时销毁事件
destroyed () {
  $(document.body).off('alarm')
}
```
3. 打包的时候统一去掉console和debugger
在项目中的build/webpack.prod.conf.js文件中
```
new webpack.optimize.UglifyJsPlugin({
  compress: {
    warnings: false,
    drop_debugger: true,
    drop_console: true
  }
```
4. computed传参（闭包）
```
:data="closure(item, itemName, blablaParams)"

computed: {
 closure () {
   return function (a, b, c) {
        /** do something */
        return data
    }
 }
}
```

