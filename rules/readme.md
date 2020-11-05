# rules

- [命名规范](#命名规范)
- [杜绝垃圾代码](#杜绝垃圾代码)
- [css规范](#css)
- [图片规范](#图片规范)
- [其余](#其余)


**规范目的**

为提高团队协作效率，便于前端后期优化维护，输出高质量的文档

**做规范的原因**

- 首先，公司项目代码不是你个人的，这点是最重要的
- 其次，每个人的风格都不一致，不可能都带着自己的风格做开发
- 开发环境和生产环境的代码有差异，项目规范工具可以在开发阶段将问题暴露出来

# 命名规范

- [项目命名](#项目命名)
- [目录名](#目录名)
- [文件名](#文件名)
- [css命名](#css命名)

## 项目命名

- 尽量一个单词，全小写
- 多个单词，以下划线分隔
- 命名要见名知意


## 目录名

参照[项目](#项目命名)命名规则；

有复数结构时，要采用复数命名法

例：scripts, styles, images, data_models

## 文件名

- `样式文件` 命名尽量以一个单词表示，多个以中横线连接
- `js` 文件命名尽量以一个单词作为名称，类函数功能文件，其名称首字母大写

## css命名

常用的 css 命名规则：

- 头：header
- 内容：content/container
- 尾：footer
- 导航：nav
- 侧栏：sidebar
- 栏目：column
- 页面外围控制整体布局宽度：wrapper
- 左右中：left right center
- 登录条：loginbar
- 标志：logo
- 广告：banner
- 页面主体：main
- 热点：hot
- 新闻：news
- 下载：download
- 子导航：subnav
- 菜单：menu
- 子菜单：submenu
- 搜索：search
- 友情链接：friendlink
- 页脚：footer
- 版权：copyright
- 滚动：scroll
- 内容：content
- 标签页：tab
- 文章列表：list
- 提示信息：msg
- 小技巧：tips
- 栏目标题：title
- 加入：joinus
- 指南：guide
- 服务：service
- 注册：register
- 状态：status
- 投票：vote
- 合作伙伴：partner

# 杜绝垃圾代码

`js` 示例
```js
var is_Hotel;
var a = 20;

function a() {
  var is_hotel;
  // ...
  // 此处省略一万行
  // ...
  console.log(a);
}
```

`css` 示例
```css
#a {
  color: #f00 !important;
}
```
- 禁止使用 `important`
- 尽量避免 `id` 样式


# css

- [css书写规范](#css书写规范)


## css书写规范

- 尽量写通用样式，达到复用
- 避免覆盖全局样式，如有同名的，通过作用域区别
- 禁止使用 `@import` 导入 `css` 样式文件到另外一个 `css` 文件
- `缩进` 使用soft tab（4个空格）
- `分号` 末尾都要加分号
- `空格` 遵循 [其余](#其余) 规范
- `空行` 遵循 [其余](#其余) 规范
- `换行` 遵循 [其余](#其余) 规范
- 公共型样式：包括了以下几个部分：“标签的重置和设置默认值”、“统一调用背景图和清除浮动或其他需统一处理的长样式”、“网站通用布局”、“通用模块和其扩展”、“元件和其扩展”、“功能类样式”、“皮肤类样式”。
- 特殊型样式：当某个栏目或页面的样式与网站整体差异较大或者维护率较高时，可以独立引用一个样式：“特殊的布局、模块和元件及扩展”、“特殊的功能、颜色和背景”，也可以是某个大型控件或模块的独立样式。
- 皮肤型样式：如果产品需要换肤功能，那么我们需要将颜色、背景等抽离出来放在这里
- `命名的顺序`
```
[
    [
        "display",
        "visibility",
        "float",
        "clear",
        "overflow",
        "overflow-x",
        "overflow-y",
        "clip",
        "zoom"
    ],
    [
        "table-layout",
        "empty-cells",
        "caption-side",
        "border-spacing",
        "border-collapse",
        "list-style",
        "list-style-position",
        "list-style-type",
        "list-style-image"
    ],
    [
        "-webkit-box-orient",
        "-webkit-box-direction",
        "-webkit-box-decoration-break",
        "-webkit-box-pack",
        "-webkit-box-align",
        "-webkit-box-flex"
    ],
    [
        "position",
        "top",
        "right",
        "bottom",
        "left",
        "z-index"
    ],
    [
        "margin",
        "margin-top",
        "margin-right",
        "margin-bottom",
        "margin-left",
        "-webkit-box-sizing",
        "-moz-box-sizing",
        "box-sizing",
        "border",
        "border-width",
        "border-style",
        "border-color",
        "border-top",
        "border-top-width",
        "border-top-style",
        "border-top-color",
        "border-right",
        "border-right-width",
        "border-right-style",
        "border-right-color",
        "border-bottom",
        "border-bottom-width",
        "border-bottom-style",
        "border-bottom-color",
        "border-left",
        "border-left-width",
        "border-left-style",
        "border-left-color",
        "-webkit-border-radius",
        "-moz-border-radius",
        "border-radius",
        "-webkit-border-top-left-radius",
        "-moz-border-radius-topleft",
        "border-top-left-radius",
        "-webkit-border-top-right-radius",
        "-moz-border-radius-topright",
        "border-top-right-radius",
        "-webkit-border-bottom-right-radius",
        "-moz-border-radius-bottomright",
        "border-bottom-right-radius",
        "-webkit-border-bottom-left-radius",
        "-moz-border-radius-bottomleft",
        "border-bottom-left-radius",
        "-webkit-border-image",
        "-moz-border-image",
        "-o-border-image",
        "border-image",
        "-webkit-border-image-source",
        "-moz-border-image-source",
        "-o-border-image-source",
        "border-image-source",
        "-webkit-border-image-slice",
        "-moz-border-image-slice",
        "-o-border-image-slice",
        "border-image-slice",
        "-webkit-border-image-width",
        "-moz-border-image-width",
        "-o-border-image-width",
        "border-image-width",
        "-webkit-border-image-outset",
        "-moz-border-image-outset",
        "-o-border-image-outset",
        "border-image-outset",
        "-webkit-border-image-repeat",
        "-moz-border-image-repeat",
        "-o-border-image-repeat",
        "border-image-repeat",
        "padding",
        "padding-top",
        "padding-right",
        "padding-bottom",
        "padding-left",
        "width",
        "min-width",
        "max-width",
        "height",
        "min-height",
        "max-height"
    ],
    [
        "font",
        "font-family",
        "font-size",
        "font-weight",
        "font-style",
        "font-variant",
        "font-size-adjust",
        "font-stretch",
        "font-effect",
        "font-emphasize",
        "font-emphasize-position",
        "font-emphasize-style",
        "font-smooth",
        "line-height",
        "text-align",
        "-webkit-text-align-last",
        "-moz-text-align-last",
        "-ms-text-align-last",
        "text-align-last",
        "vertical-align",
        "white-space",
        "text-decoration",
        "text-emphasis",
        "text-emphasis-color",
        "text-emphasis-style",
        "text-emphasis-position",
        "text-indent",
        "-ms-text-justify",
        "text-justify",
        "letter-spacing",
        "word-spacing",
        "-ms-writing-mode",
        "text-outline",
        "text-transform",
        "text-wrap",
        "-ms-text-overflow",
        "text-overflow",
        "text-overflow-ellipsis",
        "text-overflow-mode",
        "-ms-word-wrap",
        "word-wrap",
        "-ms-word-break",
        "word-break"
    ],
    [
        "color",
        "background",
        "filter:progid:DXImageTransform.Microsoft.AlphaImageLoader",
        "background-color",
        "background-image",
        "background-repeat",
        "background-attachment",
        "background-position",
        "-ms-background-position-x",
        "background-position-x",
        "-ms-background-position-y",
        "background-position-y",
        "-webkit-background-clip",
        "-moz-background-clip",
        "background-clip",
        "background-origin",
        "-webkit-background-size",
        "-moz-background-size",
        "-o-background-size",
        "background-size"
    ],
    [
        "outline",
        "outline-width",
        "outline-style",
        "outline-color",
        "outline-offset",
        "opacity",
        "filter:progid:DXImageTransform.Microsoft.Alpha(Opacity",
        "-ms-filter:\\'progid:DXImageTransform.Microsoft.Alpha",
        "-ms-interpolation-mode",
        "-webkit-box-shadow",
        "-moz-box-shadow",
        "box-shadow",
        "filter:progid:DXImageTransform.Microsoft.gradient",
        "-ms-filter:\\'progid:DXImageTransform.Microsoft.gradient",
        "text-shadow"
    ],
    [
        "-webkit-transition",
        "-moz-transition",
        "-ms-transition",
        "-o-transition",
        "transition",
        "-webkit-transition-delay",
        "-moz-transition-delay",
        "-ms-transition-delay",
        "-o-transition-delay",
        "transition-delay",
        "-webkit-transition-timing-function",
        "-moz-transition-timing-function",
        "-ms-transition-timing-function",
        "-o-transition-timing-function",
        "transition-timing-function",
        "-webkit-transition-duration",
        "-moz-transition-duration",
        "-ms-transition-duration",
        "-o-transition-duration",
        "transition-duration",
        "-webkit-transition-property",
        "-moz-transition-property",
        "-ms-transition-property",
        "-o-transition-property",
        "transition-property",
        "-webkit-transform",
        "-moz-transform",
        "-ms-transform",
        "-o-transform",
        "transform",
        "-webkit-transform-origin",
        "-moz-transform-origin",
        "-ms-transform-origin",
        "-o-transform-origin",
        "transform-origin",
        "-webkit-animation",
        "-moz-animation",
        "-ms-animation",
        "-o-animation",
        "animation",
        "-webkit-animation-name",
        "-moz-animation-name",
        "-ms-animation-name",
        "-o-animation-name",
        "animation-name",
        "-webkit-animation-duration",
        "-moz-animation-duration",
        "-ms-animation-duration",
        "-o-animation-duration",
        "animation-duration",
        "-webkit-animation-play-state",
        "-moz-animation-play-state",
        "-ms-animation-play-state",
        "-o-animation-play-state",
        "animation-play-state",
        "-webkit-animation-timing-function",
        "-moz-animation-timing-function",
        "-ms-animation-timing-function",
        "-o-animation-timing-function",
        "animation-timing-function",
        "-webkit-animation-delay",
        "-moz-animation-delay",
        "-ms-animation-delay",
        "-o-animation-delay",
        "animation-delay",
        "-webkit-animation-iteration-count",
        "-moz-animation-iteration-count",
        "-ms-animation-iteration-count",
        "-o-animation-iteration-count",
        "animation-iteration-count",
        "-webkit-animation-direction",
        "-moz-animation-direction",
        "-ms-animation-direction",
        "-o-animation-direction",
        "animation-direction"
    ],
    [
        "content",
        "quotes",
        "counter-reset",
        "counter-increment",
        "resize",
        "cursor",
        "-webkit-user-select",
        "-moz-user-select",
        "-ms-user-select",
        "user-select",
        "nav-index",
        "nav-up",
        "nav-right",
        "nav-down",
        "nav-left",
        "-moz-tab-size",
        "-o-tab-size",
        "tab-size",
        "-webkit-hyphens",
        "-moz-hyphens",
        "hyphens",
        "pointer-events"
    ]
]
```

# js规范

- [js命名规范](#js命名规范)
- [js注释规范](#js注释规范)
- [API文档规范](#API文档规范)
- [继承](#继承)
- [js特别容易在线上出问题的写法](#js特别容易在线上出问题的写法)

## js命名规范

常用命名规范
```
get 获取/set 设置,
add 增加/remove 删除
create 创建/destory 移除
start 启动/stop 停止
open 打开/close 关闭,
read 读取/write 写入
load 载入/save 保存,
create 创建/destroy 销毁
begin 开始/end 结束,
backup 备份/restore 恢复
import 导入/export 导出,
split 分割/merge 合并
inject 注入/extract 提取,
attach 附着/detach 脱离
bind 绑定/separate 分离,
view 查看/browse 浏览
edit 编辑/modify 修改,
select 选取/mark 标记
copy 复制/paste 粘贴,
undo 撤销/redo 重做
insert 插入/delete 移除,
add 加入/append 添加
clean 清理/clear 清除,
index 索引/sort 排序
find 查找/search 搜索,
increase 增加/decrease 减少
play 播放/pause 暂停,
launch 启动/run 运行
compile 编译/execute 执行,
debug 调试/trace 跟踪
observe 观察/listen 监听,
build 构建/publish 发布
input 输入/output 输出,
encode 编码/decode 解码
encrypt 加密/decrypt 解密,
compress 压缩/decompress 解压缩
pack 打包/unpack 解包,
parse 解析/emit 生成
connect 连接/disconnect 断开,
send 发送/receive 接收
download 下载/upload 上传,
refresh 刷新/synchronize 同步
update 更新/revert 复原,
lock 锁定/unlock 解锁
check out 签出/check in 签入,
submit 提交/commit 交付
push 推/pull 拉,
expand 展开/collapse 折叠
begin 起始/end 结束,
start 开始/finish 完成
enter 进入/exit 退出,
abort 放弃/quit 离开
obsolete 废弃/depreciate 废旧,
collect 收集/aggregate 聚集
```

## js注释规范

>为代码编写注释是非常重要的。通常注释标注这段代码的工作原理。但是当过一周后再次回到该代码时，可能会花上很长时间来回想起那段代码到底是干什么的。

- 公共组件维护者和各栏目WD都需要在文件头部加上注释说明
```js
/**
 * 文件用途说明
 * 作者姓名、联系方式（旺旺）
 * 制作日期
 **/
```
- 多行注释
```js
//================
// 代码用途
//================
```
- 单行注释
```
// 你的注释
```

## API文档规范

- 为API编写注释不仅仅是一中提供参考文档的简便方法，而且还有其他用途——通过再次审查代码，提高代码质量。
- 在解决问题时写出的解决方案仅仅是一个初稿。该解决方案可以给出令人期待的输出，但是该方案是否是最佳方案呢？改代码是否可读、易于理解、维护和升级呢？当您再次审视代码时您将更加确定代码哪些部分可以改进——如何使得代码更容易继续更新，移除一些不足之处等。它可以极大地帮助您创建高质量的代码。


## 继承

关于继承，有大量文档知识。建议一个类包含一个完整的功能。

>推荐使用[`组合寄生式继承`](https://github.com/sileny/docs/tree/master/js/extends)

## js特别容易在线上出问题的写法

- 没有关闭标记，只是缩进，<s>`python` 风格代码</s>
```js
// 不推荐的书写
 if (true)
     // coding...
```
- 禁止使用 `eval`，如果非要用，建议使用 `new Function`，但是，它们都会加大开销因为每一次脚本引擎调用他们是必须将源码转换成可执行代码
- 句尾没有分号，引擎需要推断语句是否结束，增加解析成本
- `window.onload` 只能使用一次，使用多次会被最后的覆盖
- 过多的定时器，定时器嵌套，建议使用 `raf`
- 尽量不要使用 `with`
- 避免使用全局变量
- 避免在对性能影响较大的地方使用 `for-in`，它会从头遍历到尾
- 尽量少的使用改变元素尺寸或位置的操作，页面重新渲染代价昂贵，尤其是首屏


暂只列举几种

# 图片规范

- 每个模块都会增加一个图片文件夹，方便后期维护和处理，请将同一个模块的图片放在同一个文件夹里面，图片文件命名尽量跟css文件的命名相同，尽量使用小写命名。
- 图片格式仅限于gif || png || jpg;
- 命名全部用小写英文字母 || 数字 || -(统一用分划线衔接) 的组合，其中不得包含汉字 || 空格 || 特殊字符；尽量用易懂的词汇, 便于团队其他成员理解; 另, 命名分头尾两部分, 用分划线隔开, 比如ad-left01.gif || btn-submit.gif;
- 在保证视觉效果的情况下选择最小的图片格式与图片质量, 以减少加载时间;
- 尽量避免使用半透明的png图片(若使用, 请参考css规范相关说明);
- 运用css sprite技术集中小的背景图或图标, 减小页面http请求。


# 其余

遵循项目配置规范

- eslint
- [prettier](https://prettier.io/) 支持多语言、多框架书写规范
- commit规范

`eslint` 配置项非常多，可以参考[官网](https://eslint.org/docs/user-guide/)
