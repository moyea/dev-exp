# emeritus-platform-web

## 项目说明

emeritus管理后台，包括官网展示资源(学校，班级，班次)管理，用户相关(申请，订单，学籍，转班等)管理，部分官网页面(课程，问卷等)自定义，也包括一些系统固定参数等相关管理

## 项目对应地址

- 测试环境 platform.ushi.cn
- 正式环境 platform.emeritus.plus

## 开发说明

项目采用 [nextjs 13](https://nextjs.org/) + antd搭建，使用的是nextjs 13的app route作为路由，详细信息可参阅官方文档。需要注意的是，**由于登录全部走account，而account无法跳回localhost，所以需要先对应环境走一遍登录，然后将platform相关session（`_EMER_PLATFORM_`开头）的Domain更改为localhost才能实现本地连接到对应的环境。**这里以chrome本地调试测试环境为例：

1. 本地npm run dev 启动项目
2. 浏览器访问platform.ushi.cn -->  跳转到account.ushi.cn登录页 -->   完成登录后跳转回platform.ushi.cn 
3. 打开chrome开发者工具 -> Application -> Cookies,   找到 `_EMER_PLATFORM_`开头的几个cookie，将其Domain更改为localhost
4. 访问第一步中本地的地址，之后可以使用测试环境的数据来进行调试

登录检查，以及页面权限检查均在`src/middleware.ts` 中完成

## 页面权限说明

platform每个页面的权限都包含一个page权限(对应portal中权限类型为`page`，值为页面地址，如果是动态页面，则为不包含动态值的前缀部分)，一个或多个api权限（对应portal权限类型为`uri`，值为`/api`开头），以及多个按钮权限(由于按钮多半涉及到跳转，这里就简化为跳转后页面相关权限有无去做判断了，未单独设置新的权限类型)三部分组成。相关代码可参阅：

- page权限：页面权限校验`src/middleware.ts`, 用户左侧显示的菜单 `src/ui/dashboard.tsx` 
- api权限：前端大部分用来作为按钮权限的判断，更多的是后端去使用
- 按钮权限： `src/auth`中`AuthWrapper`以及`useAuth`

## 项目目录

```
.
├── ./public  // 资源文件
├── ./src     // 代码源文件
│   ├── ./src/app // 页面路由
│   ├── ./src/assets // 资源文件
│   ├── ./src/auth   // 按钮权限控制
│   ├── ./src/components // 部分常用组件
│   ├── ./src/config     // 项目配置文件
│   │   ├── ./src/config/config.dev.js  // 测试环境及本地开发环境配置
│   │   ├── ./src/config/config.local.js // 暂未使用
│   │   ├── ./src/config/config.prod.js  // 生产环境 
│   │   ├── ./src/config/index.js        // 配置文件
│   │   ├── ./src/config/server.config.js 
│   │   └── ./src/config/version.js      // 版本记录文件
│   ├── ./src/constants   // 项目常量
│   ├── ./src/contexts    // react部分自定义context
│   ├── ./src/hooks       // 自定义hooks
│   ├── ./src/icons       // 部分图标文件
│   ├── ./src/middleware.ts // nextjs的全局middleware
│   ├── ./src/services      // 后端接口文件
│   │   ├── ./src/services/api.ts // 自定义axios配置
│   │   └── ./src/*.ts      // 后端服务对接相关ts
│   ├── ./src/styles        // 样式文件
│   ├── ./src/types         // ts的type定义文件
│   ├── ./src/ui            // 所有ui相关
│   └── ./src/utils         // 工具类
├── ./tsconfig.json
└── ./yarn.lock
```

## 部署及发布

项目采用github actions来自动化部署，具体可参见项目根目录 `./github/workflows/deploy_uat_platform_web.yml`, `./github/workflows/deploy_prd_platform_web.yml`中相关配置。

因此，只需要在分支上打上`uat-v*`, `prd-v*` 并推送到github，即可完成测试环境及生产环境的自动化打包，需要注意，生产环境需同时更改`src/config/version.js`中记录的版本信息。另外为防止代码混乱，一般只在dev分支上打`uat-v*`相关tag，main分支上打`prd-v*` 相关tag。所以一般是在dev测试通过后，合并到main分支，然后更新main分支上`src/config/version.js`中的版本信息，之后打上`prd-v*` 的tag即可完成发布。

## 配置文件说明

`src/config/config.prod.js`

```javascript
module.exports = {
  // 项目访问的接口地址，本地开发中nextjs代理时及部分情况下会使用，
  apiHost: 'https://platform.emeritus.plus', 
  // 项目中登录验证的地址，具体使用参阅src/middleware.ts
  authHost: 'https://account.emeritus.org.cn',
  // 官网adcarry地址，部分情况下需要前端拼接一个页面地址供内部查看
  interface: 'https://emeritus.org.cn', //官网地址
  // 在portal项目中生成的应用id，用于account跳转及相关校验
  clientId: '1a2cb35d1f4448afbd0a28fb94a5e120',
  // ga
  googleTagId: 'G-NQM60XXWND',
  // 神策，未使用
  sensorUrl: 'https://matrix.emeritus.org.cn/sa?project=production'
};
```

## 其他特殊说明

生产环境和测试环境需通过 读取`process.env.SERVICE_ENV`来进行区分，所以若发现环境异常，需检查部署的服务器上`SERVICE_ENV`是否为`dev`或`prod`



# emeritus-account-mng

## 项目说明

主要是内部应用，以及人员相关菜单权限，接口权限，按钮权限的管理

## 项目对应地址

- 测试环境 portal.ushi.cn
- 正式环境 portal.emeritus.org.cn

## 开发说明

纯react+antd项目，项目结构比较简单，登录也是跳转account，可参考 emeritus-platform-web 项目进行本地开发

## 部署及发布

项目采用github actions来自动化部署，具体可参见项目根目录 `./github/workflows/deploy_uat_account_mng.yml`, `./github/workflows/deploy_prd_account_mng.yml`中相关配置。

只需要在分支上打上`uat-v*`, `prd-v*` 并推送到github，即可完成测试环境及生产环境的自动化打包

## 重要文件说明

`src/router/permission.ts` 文件中包含登录相关校验及跳转



# emeritus-account-web

## 项目说明

emeritus前端项目集中单点登录项目

## 项目地址

* 测试地址： https://account.ushi.cn/
* 正式地址：https://emeritus.org.cn/

## 开发说明

nextjs13+antd项目，使用的是nextjs 13的page route作为路由

## 其他项目接入指南

具体前端代码实现可参考如下代码：

```typescript
/* 用户访问sourceURI，接入项目检查用户有效，无效的情况下跳转至account（也就是下面的url地址）进行登录，account登录处理完成之后携带ticket跳转至redirectURI（也就是后端回调地址），后端通过ticket拿到用户信息之后，将相关cookie写入，同时解析state，并重定向至sourceURI完成登录处理，具体 /api/v5/login/callback 的实现，可参考后端代码 */
// 登录认证地址
const AuthHost = 'https://account.ushi.cn';
// 接入项目的后端地址
const APIHost = 'https://platform.ushi.cn';
// 由portal创建的clientId
const ClientId = 'bcf81992f4e14f5ca7a2c1ec575e43f7';
// 接入项目的后端回调地址，account登录完成后会携带ticket请求该回调地址，由后端将用户相关的cookie写入客户端
const redirectURI = encodeURIComponent(`${APIHost}/api/v5/login/callback`)
// 用户访问的目标地址
const sourceURI = encode(btoa('https://platform.ushi.cn/account/list')))
// 拼装好的登录认证地址
const url = `${AuthHost}/login?response_type=code&client_id=${ClientId}&redirect_uri=${redirectURI}&scope=full&state=${sourceURI}`
// 跳转到登录认证
window.location.href= url;
```

## 部署及发布

项目采用github actions来自动化部署，具体可参见项目根目录 `./github/workflows/deploy_uat_account_web.yml`, `./github/workflows/deploy_prod_account_web.yml`中相关配置。

因此，只需要在分支上打上`uat-v*`, `prd-v*` 并推送到github，即可完成测试环境及生产环境的自动化打包。另外，**生产环境需注意`src/config/index.js`中`isProd` 的值是否为`true`。**

# emeritus_adcarry_web

## 项目说明

emeritus C端官网项目，包括课程展示，申请，支付，订单展示，问卷以及各种表单功能。登录相关实现时单点登录，具体实现可参考 emeritus-account-web 项目

## 项目地址

* 测试环境：https://uat2.ushi.cn/   ，对应开发分支为uat2
* 生产环境：https://emeritus.org.cn/ ，对应分支为main

## 开发说明

Nextjs12 + antd实现的ssr项目，以page route作为项目路由。

## 部署及发布

项目采用github actions来自动化部署，具体可参见项目根目录 `./github/workflows/deploy_uat2_adcarry_web.yml`, `./github/workflows/deploy_prd_adcarry_web.yml`中相关配置。

因此，只需要在分支上打上`uat2-v*`, `prd-v*` 并推送到github，即可完成测试环境及生产环境的自动化打包。测试环境也可以直接推送至uat2分支，不打tag也能完成测试环境的自动化打包。

## 配置文件说明

`src/config/config.prod.js`

```javascript
module.exports = {
  // 项目访问的接口地址，本地开发中nextjs代理时及部分情况下会使用，
  apiHost: 'https://emeritus.org.cn', 
  // 项目中登录验证的地址，具体使用参阅src/middleware.ts
  authHost: 'https://account.emeritus.org.cn',
  // 官网adcarry地址，部分情况下需要前端拼接一个页面地址供内部查看
  interface: 'https://emeritus.org.cn', //官网地址
  // 在portal项目中生成的应用id，用于account跳转及相关校验
  clientId: '27bc8093d2c042bfadbfba6a92f0523d',
  // ga
  googleTagId: 'G-EFXMQEPS8T',
  // 神策，未使用
  sensorUrl: 'https://matrix.emeritus.org.cn/sa?project=production'
};
```

## 其他特殊说明

生产环境和测试环境需通过 读取`process.env.SERVICE_ENV`来进行区分，所以若发现环境异常，需检查部署的服务器上`SERVICE_ENV`是否为`dev`或`prod`

