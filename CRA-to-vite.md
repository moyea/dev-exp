随着vue3的发布，带来了新的打包工具Vite，作为前端的一员，自然得体验一番。由于工作中react用的比较多，创建项目基本都是通过 `create-react-app` （CRA）来创建。那么通过CRA创建react项目的怎么迁移到vite呢？以下将带大家了解一下如何将一个react项目从CRA迁移到vite:

## 初始设置

打开一个通过CRA创建的react项目

首先安装vite:

```
// npm
npm install vite

// yarn
yarn add vite
```

然后更新 npm scripts:

```diff
"scripts": {
-    "start": "react-scripts start",
-    "build": "react-scripts build",
+    "start": "vite",
+    "build": "vite build",
     "test": "react-scripts test",
```

接下来需要修改 `public/index.html` 文件，原因[官方文档](https://cn.vitejs.dev/guide/#community-templates)有说明

将`./public`目录下`index.html`移到根目录下，同时添加script标签，src指向入口文件，如果html中有任何`%PUBLIC_URL%/`，将它改为`/`。原因官方上面的文档中有说明。以下是我这边的改动

```diff
diff --git a/public/index.html b/index.html
similarity index 88%
rename from public/index.html
rename to index.html
index aa069f2..dd43c8b 100644
--- a/public/index.html
+++ b/index.html
@@ -2,19 +2,19 @@
 <html lang="en">
   <head>
     <meta charset="utf-8" />
-    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
+    <link rel="icon" href="/favicon.ico" />
     <meta name="viewport" content="width=device-width, initial-scale=1" />
     <meta name="theme-color" content="#000000" />
     <meta
       name="description"
       content="Web site created using create-react-app"
     />
-    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
+    <link rel="apple-touch-icon" href="/logo192.png" />
     <!--
       manifest.json provides metadata used when your web app is installed on a
       user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
     -->
-    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
+    <link rel="manifest" href="/manifest.json" />
     <!--
       Notice the use of %PUBLIC_URL% in the tags above.
       It will be replaced with the URL of the `public` folder during the build.
@@ -29,6 +29,7 @@
   <body>
     <noscript>You need to enable JavaScript to run this app.</noscript>
     <div id="root"></div>
+    <script type="module" src="/src/index.tsx"></script>
     <!--
       This HTML file is a template.
       If you open it directly in the browser, you will see an empty page.
```

### Typescript 支持

到目前为止，我的项目还没法run起来。主要是vite并不理解`tsconfig.json` 中 `include` 属性使用的绝对路径导入。解决方法是安装一个`vite-tsconfig-paths`的插件来支持typescript的[路径映射](https://www.typescriptlang.org/docs/handbook/module-resolution.html#path-mapping) 。

```shell
// npm
npm install vite-tsconfig-paths

// yarn
yarn add vite-tsconfig-paths
```

安装完成后，在根目录添加`vite.config.ts`

```typescript
import { defineConfig } from "vite";
import tsConfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  build: {
    outDir: "build"
  },
  plugins: [tsConfigPaths()]
});
```

这里我们的输出目录设置为`build`，而不是默认的`dist`，是为了和CRA保持一致。

现在我们的项目就可以通过vite跑起来了。

### Jest 支持

由于我们准备从项目中移除`react-scripts`，所以我们需要解决怎么不通过`react-scripts test`来运行Jest测试。

我们需要借助于`ts-jest`，添加依赖

```shell
// npm
npm install ts-jest

// yarn
yarn add ts-jest
```

然后再`jest.config.js`中添加`ts-jest`的preset:

```javascript
module.exports = {
  preset: "ts-jest",
  setupFilesAfterEnv: ["<rootDir>src/setupTests.ts"],
  moduleDirectories: ["node_modules", "src"],
  moduleNameMapper: {
    "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/tests/__mocks__/fileMock.js",
    "\\.(css|less|scss|sass)$":"<rootDir>/tests/__mocks__/styleMock.js"
  }
}
```

注意，由于react项目中可以import css/png之类的文件，但是jest会将其视为js导入，会导致测试失败，所以我们需要针对这种情况做处理，就是`moduleNameMapper`所做的，参考 [Jest处理静态资源](https://jestjs.io/docs/webpack#handling-static-assets)。

在`tests/__mocks__/`目录下添加`fileMock.js`和`styleMock.js`

```jsvascript
// fileMock.js
module.exports="test-file-stub"
```

```javascript
// styleMock.js
module.exports = {}
```

更新package.json

```diff
"scripts": {
    "start": "vite",
    "build": "vite build",
-   "test": "react-scripts test",
+   "test": "jest",
```

### 收尾工作

既然我们已经将react项目通过vite来构建了，那么CRA相关的依赖也都可以移除了，我这边只需要移除react-scripts就可以了，实际需要根据自己的项目来确定

```shell
// npm
npm uninstall react-scripts

// yarn
yarn remove react-scripts
```

### 总结

整体下来我们做了一些工作，在开发环境，通过vite来启动整体快了很多。

关于为什么用vite，官方文档[为什么选vite](https://cn.vitejs.dev/guide/why.html)中已经做出了说明。

但是，毕竟vite还是一个很新的工具，新就意味着解决方案并不是很多，生态还不够成熟，潜在许多坑等等。

作为学习，我认为选择什么工具都可以，但是实际工作中，还是需要选择更加稳定，生态更加丰富的工具。