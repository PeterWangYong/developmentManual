## 简介

Express是基于NodeJS开发的Web框架。

```js
# npm i -S express

# 创建应用对象
const express = require('express')
const app = express()

# 注册各种中间件
app.use(...)
app.use(...)

# 启动服务器
app.listen(5000)
```



## 三大概念

###中间件

中间件是一个函数，Express中所有的逻辑处理都依赖于多个中间件的顺序调用。

```###js
app.use(function(req, res, next) {
  console.log('midware one')
  next()
})

app.use(function(req, res, next) {
  console.log('midware two')
  res.send('finish')
})

# 1. app.use() 注册全局中间件
# 2. 通过req, res 获取请求参数并返回响应，单次请求只能有一次响应
# 3. next() 调用下一个中间件，中间件调用顺序取决于注册顺序
# 4. 除了自定义中间件，还有express内置中间件和第三方中间件
```

###路由

路由通过匹配请求路径决定使用哪个中间件处理请求。

```js
app.use('/one', function(req, res, next) {
  console.log('midware one')
  res.send('handle one')
})

# 1. app.use('/one', ...) 添加路径参数，表示只有当路径匹配成功才执行该中间件

app.get('/two', function(req, res, next) {
  console.log('midware two')
  res.send('handle two')
})

# 2. app.get('/two', ...) 在app.use()基础上限定了请求方法，类似的还有 app.post() ...

const router = express.Router()
router.use(...)
router.use('/one', ...)
router.get('/two', ...)

# 3. router是一个子路由，也是一个中间件，类似小型的app对象
# 4. router注册的中间件只在该router内部有效
           

app.use('/user', router)

# 5. 将router注册到全局中间件，通常用于路由分组
# 6. 需要访问/user/xxx才能调用到router中的中间件
```

###异常处理

异常处理需要用到一个特殊的中间件，称为异常处理器(errorHandler)。异常处理中间件比普通中间件多接收一个err参数，通常放置在最后注册，可以捕获全局抛出的异常或者next传递过来的参数。express默认有一个异常处理器用于异常处理。

```js
app.use(function(req, res, next) {
  throw new Error('error')
})

app.use(function(err, req, res, next) {
 	console.log(err.message)
})

# 1. 异常处理器可以捕获throw抛出的错误

app.use(function(req, res, next) {
  next(new Error('error'))
})

app.use(function(err, req, res, next) {
 	console.log(err.message)
})

# 2. 异常处理器可以接收next(err)传递过来的参数，next()调用普通中间件，next(err)调用异常处理器

```



## 项目：登录鉴权

### 需求说明

/user/login： 前台传入用户名密码，后台验证成功后返回token保存状态

/user/info： 前台携带token请求用户信息，后台校验token成功后返回数据

### 前置组件

项目中需要用到一些第三方组件

#### body-parser

用于解析post请求体中的数据,解析结果自动挂到req对象中

```js
const bodyParser = require('body-parser')
app.use(bodyParser.urlencoded({extended: true})) # 解析x-www-form-urlencoded类型数据
app.use(bodyParser.json()) # 解析json类型数据
```

#### jsonwebtoken

生成和校验JsonWebToken（JWT是一种Token规范）

```js
const jwt = require('jsonwebtoken')
# 签发token:三个参数为：自定义payload，加密私钥，标准payload
jwt.sign({username: 'xxx'}, privateKey, {expiresIn: '1h'}) 
# 校验token:返回payload对象
jwt.verify(token, privateKey)
```

####express-jwt

创建中间件用于JWT的校验，默认按照Authorization: Bearer xxxxx的格式获取

```js
const expressJwt = require('express-jwt')
const jwtAuth = expressJwt({
  secret: privateKey, 	# 加密私钥
  credentialsRequired: true  # 开启校验
}).unless({
  path: [
    '/user/login'		# 部分路径不用校验
  ]
})
app.use(jwtAuth)

```

#### boom

针对常见HTTP错误返回响应的数据对象，包括状态码，状态值，错误信息，Error对象等

```js
const boom = require('boom')
const notFound = function(req, res, next) {
  next(boom.notFound('资源不存在')) # 使用next(...)配合错误处理器完成统一异常处理
}
```

> 安装方式：npm install xxx



### 项目主体

1. 首先创建入口文件server.js，接着往里面填充中间件

```js
const express = require('express')
const app = express()

app.use('/', indexRouter)

const server = app.listen(5000, function() {
  const {address, port} = server.address()
  console.log(`listening at ${address}:${port}`)
})
```

2. 接着创建routes/user.js用于构建请求处理中间件

```js
router.post('/login', function(req, res) {} 	# 如果用不到next可以不接收
router.get('/info', function(req, res) {}

# 另外创建routes/index.js 作为父级路由,便于server.js中注册
const router = express.Router()
router.use('/user', userRouter)
```

3. /user/login逻辑处理

```js
if (username === 'admin' && password === 'password') {
  	# 校验用户名密码并返回token
    const token = jwt.sign({username}, privateKey, {expiresIn: '1h'})
    res.send({code: 0, msg: 'success', data: {token}})
} else {
  	res.send({code: 1000, msg: '用户名密码错误'})
}
```

4. /user/info逻辑处理

```js
# 从token中获取到用户名，然后通常会根据用户名查数据库返回用户信息，这里省略查询数据库过程
const payload = jwtDecode(req)
res.send({code: 0, msg: 'success', data: {
    username: payload.username,
    roles: ['admin']
  }
})

#  jwtDecode是封装的函数，从请求中获取token然后解析返回payload数据
function jwtDecode(req) {
  const token = req.get('Authorization').replace('Bearer ', '')
  return jwt.verify(token, privateKey)
}
```

### 细节说明

上述的主体逻辑基本就是在构建两个请求处理中间件，然后将中间件注册到router和app。除此之外，项目中还需要一些其他的中间件保证项目的完整性。

1. 使用body-parser解析请求体数据

```js
app.use(bodyParser.urlencoded({extended: true}))
app.use(bodyParser.json())
```

2. 使用express-jwt完成自动鉴权

```js
const jwtAuth = expressJwt({
  secret: privateKey,
  credentialsRequired: true
}).unless({
  path: [
    '/user/login'
  ]
})

app.use(jwtAuth)
```

3. 构建一个notFound处理中间件

```js
const notFound = function(req, res, next) {
  next(boom.notFound('资源不存在'))
}

app.use(notFound)
```

4. 构建一个错误处理器

```js
const errorHandler = function(err, req, res, next) {
  if (err.name === 'UnauthorizedError') {
    res.status(err.status).send({
      code: -2,
      msg: 'Token不存在或者已过期'
    })
  } else {
    const status = err.output && err.output.statusCode || 500
    res.status(status).send({
      code: -1,
      msg: (err && err.message) || '未知错误'
    })
  }
}

app.use(errorHandler)
```

### 整体回顾

回顾整个项目，我们发现核心就是这么几个中间件：

```js
app.use(bodyParser.urlencoded({extended: true}))
app.use(bodyParser.json())
app.use(jwtAuth)

app.use('/', indexRouter)

app.use(notFound)
app.use(errorHandler)

```

其他的都是项目层面的分目录结构，函数封装等，所以还是比较清晰的。



## 总结

Express是NodeJS中比较知名和老牌的Web框架，通过中间件模式构建逻辑层次（中间件模式【洋葱模型】是比较典型的逻辑层次模型，在Python，JS等动态语言中使用广泛），使用简单，易于上手。但是Express的缺点在于其中间件函数使用的是回调模式，大量的回调不利于代码的阅读和维护，因而Express的原班人马又打造了Koa框架，Koa基于Promise模式，可以使用async和await进行同步模式开发，现在逐渐成为主流。

> 源代码：https://github.com/PeterWangYong/blog-code/tree/master/express

