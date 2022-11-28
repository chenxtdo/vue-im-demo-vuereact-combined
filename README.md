# vue-im-demo
> vue 下 imuikit 跟 callkit demo
![demo](./demo.png)

相关组件是基于React开发的，为了方便vue开发者使用，demo 使用了 [vuereact-combined](https://github.com/devilwjp/vuereact-combined) 进行了组件包装，具体参考示例代码。（PS：vuereact-combined 只适用于 Vue2 的项目，如果项目使用了 Vue3 需要把 vuereact-combined 替换为 [veaury](https://github.com/devilwjp/veaury)）
## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```


##  使用
请在 `App.vue` 文件中，相关注释下添加用户自己的配置
![config](./config.png)
### 注意
**请注意 `@xkit-yx/im-kit-ui@0.2.0` 版本，只适用呼叫组件 `@xkit-yx/call-kit-react-ui@0.1.7 @xkit-yx/call-kit@1.6.5` 版本**
<br/>
具体参考 `package.json` 下的版本依赖


