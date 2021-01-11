## 1. 统一认证和单点登录 流程

流程图

![img](http://processon.com/chart_image/5fbca9516376894d8c61d0c2.png)

client 视角流程图

![](http://processon.com/chart_image/5ff7f77f07912930e02478ff.png)

## 2.统一登出 流程

![img](http://processon.com/chart_image/5f3dd4c95653bb06f2d94759.png)

## 3. 接入改造流程

### 前置步骤：

合法注册，成为idc 的受信客户端，获取 clientId 和clientSecret

获取 IdC 的服务器信息：{idc_domain}/.well-known/idc_configuration

获取 RSA 公钥信息:     {idc_domain}/.well-known/jwks.json

### 步骤1：

改造 登录拦截器，检测到用户未登录，那么需要重定向到 idc的authorize 页面

- 如果是UA 直接访问，直接重定向

- 如果是Ajax 请求，那么 response 返回403 + redirectUrl = idc的authorize 页面，js 接收后跳转到该redirectUrl



### 步骤2:

增加 回调接口。该接口用于 在用户完成认证后，接收idc 的认证完成信息

回调接口的处理逻辑：

1. 发起授权请求
2. 获得用户授权，idc 返回授权码
3. 根据授权码 换 token/id_token，并缓存令牌
4. 根据令牌 解析或者 请求获取用户信息
5. 获取用户信息
6. 构建自身session

### 步骤3:

改造 登出接口，在原登出的基础上，同时需要通知IdC 

处理逻辑：

1. 从 缓存/session 中获取令牌
2. 清理 缓存/session
3. 用令牌做参数，重定向到IdC 的end_session 页面
4. 增加登出成功的页面（或者统一跳转IdC的登录页面）



第4步，如果业务系统自身需要由登出提示页面，可以在注册时填入，该页面由业务系统实现。否则就跳转到IdC的登录页面

### 步骤4:

增加 被动登出的接口，用于IdC 登出时，接收IdC的登出 通知，验证请求后，清理自身的 缓存/session

处理逻辑：

1. IdC发出登出通知，会转由UserAgent 触发 登出请求
2. 接收到登出请求，对请求参数进行验证
3. 验证通过后，清理自身缓存/session
4. 用户再次访问时，无session，经过步骤1 回到登录页面



## 3.工程配套

引入maven 依赖

```
<dependency>
    <groupId>com.polarquant</groupId>
    <artifactId>idc-client-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

增加系统配置

```
idc.client.client-id=YOUR_CLIENT_ID
idc.client.client-secret=YOUR_CLIENT_SECRET
idc.client.scope=openId,profile,email,phone
idc.client.grant-type=authorization_code
idc.client.response-type=id_token
idc.client.response-mode=query
#client-callback-end-point
idc.client.redirect-urls=http://idc-client.xyz/idc/login
#idc-server-issuer
idc.server.issuer=http://idc-server.xyz/idc
idc.server.authorization-uri=${idc.server.issuer}/authorize
idc.server.token-uri=${idc.server.issuer}/token
idc.server.end-session-uri=${idc.server.issuer}/end_session
idc.server.jwk-set-uri=${idc.server.issuer}/.well-known/jwks.json
idc.resource.user-info-uri=${idc.server.issuer}/user_info
```
