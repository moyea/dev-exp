## 使用node下载网络图片到本地

### 1.添加相关依赖

```shell
yarn add request
```

### 2.代码实现

```javascript
const fs = require('fs')
const request = require('request')
const path = require('path')

const tempPath = path.join(__dirname, 'temp', 'images')
// fs.createWriteStream不会自动创建父级目录，缺少该步骤，可能会报文件或目录找不到
fs.mkdirSync(tempPath, { recursive: true })
const url =
  'https://image.shutterstock.com/image-photo/beautiful-tiger-isolated-on-white-600w-186295301.jpg'
const filename = url.split('/').pop()

request(url).pipe(fs.createWriteStream(path.join(tempPath, filename)))
```

