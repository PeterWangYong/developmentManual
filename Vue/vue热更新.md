@vue/cli 构建项目

```js
vue.config.js

module.exports = {
    devServer: {
        hot: true,  // 热更新
        open: true, // 编译完成自动打开浏览器
        port: 8080,
        host: "127.0.0.1"
    }
}
```

