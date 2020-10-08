# safety

- [生产环境报错信息二次处理](#error-handler)
- [cookie设置](#cookie)

### error-handler

在生产环境下，将报错信息二次处理后输出，避免敏感信息暴露出去，[参考这里](https://github.com/sileny/node-demo/tree/main/connect-demo#error-handler)

### cookie

- 设置 `httpOnly` 为 `true`，只可以在服务端操作，客户端js无法操作，可以防止 `xss` 攻击
- `secure` 设置为 `true`，需要 `https` 支持，在 `http` 下不支持
- `signed` 设置为 `true`，拒绝未签名的 `cookie`

