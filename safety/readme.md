# safety

- [生产环境报错信息二次处理](#error-handler)
- [cookie设置](#cookie)
- [mongo结合express-session会话缓存多重校验](#session)
- [body-parser限制请求体大小](#limit)

### error-handler

在生产环境下，将报错信息二次处理后输出，避免敏感信息暴露出去，[参考这里](https://github.com/sileny/node-demo/tree/main/connect-demo#error-handler)

### cookie

- 设置 `httpOnly` 为 `true`，只可以在服务端操作，客户端js无法操作，可以防止 `xss` 攻击
- `secure` 设置为 `true`，需要 `https` 支持，在 `http` 下不支持
- `signed` 设置为 `true`，拒绝未签名的或被篡改的 `cookie`，可以有效防止 `中间人` 攻击。[案例参考](https://github.com/sileny/node-demo/tree/main/web-express#cookie-parser)

### session

结合数据库和会话实现多重认证校验

- [mongodb结合session实现认证](https://github.com/sileny/node-demo/tree/main/session-mongodb)
- [redis结合session以及对cookie签名实现认证](https://github.com/sileny/node-demo/tree/main/session-redis)
