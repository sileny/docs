# cdn

写好的代码可以发布到 `npm` 仓库，也可以发布到 `github` 仓库

- [npm](#npm)
- [unpkg](#unpkg)
- [github](#github)

## npm

`npm publish --access public`

直接就可以访问 `cdn` 资源

```
https://cdn.jsdelivr.net/gh/sileny/unity@1.0.0/dist/index.min.js

https://cdn.jsdelivr.net/npm/compress-base64@1.0.2/dist/index.min.js
https://cdn.jsdelivr.net/npm/compress-base64
```


## unpkg

```
https://unpkg.com/@silen/unity@1.0.0/dist/index.js

https://unpkg.com/compress-base64@1.0.2/dist/index.js
https://unpkg.com/compress-base64
```

`unpkg` 是一个快速的全球内容交付网络，适用于npm上的所有内容


## github

点击 `release`，填写 版本号、发布的标题 等信息后，点击 `publish` 按钮，然后，访问如下地址即可看到发布的资源

```
https://cdn.jsdelivr.net/gh/sileny/unity@1.0.0/dist/index.min.js
```


# more

访问 [jsdelivr](https://www.jsdelivr.com/)
