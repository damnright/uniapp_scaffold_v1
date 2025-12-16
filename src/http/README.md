# 请求库

目前unibest支持2种请求库：
- 菲鸽简单封装的 `简单版本http`，路径（src/http/http.ts），对应的示例在 src/api/foo.ts
- `vue-query`, 路径（src/http/vue-query.ts）, 目前主要用在自动生成接口，详情看(https://unibest.tech/base/17-generate)，示例在 src/service/app 文件夹

## 如何选择
如果您以前用过 vue-query，可以优先使用您熟悉的。
如果您的项目简单，简单版本的http 就够了，也不会增加包体积。