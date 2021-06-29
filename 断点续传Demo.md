# 断点续传Demo

```javascript
// server.js
const http = require('http')
const static = require('node-static')
const fileServer = new static.Server('.')
const path = require('path')
const fs = require('fs')
const debug = require('debug')('example:resume-upload')

const uploads = Object.create(null)

function onUpload(req, res) {
  const fileId = req.headers['x-file-id']
  const startByte = +req.headers['x-start-byte']

  if (!fileId) {
    res.writeHead(400, 'missing file Id')
    res.end()
  }

  const filePath = path.join('/tmp', fileId)

  debug('onUpload fileId: ', fileId)

  if (!uploads[fileId]) uploads[fileId] = {}

  let upload = uploads[fileId]

  debug(`bytesReceived: ${upload.bytesReceived} startByte: ${upload.startByte}`)

  let fileStream

  if (!startByte) {
    upload.bytesReceived = 0

    fileStream = fs.createWriteStream(filePath, { flags: 'w' })

    debug(`New file created: ${filePath}`)
  } else {
    if (upload.bytesReceived !== startByte) {
      res.writeHead(400, 'Wrong start byte')
      res.end(upload.bytesReceived)
      return
    }

    fileStream = fs.createWriteStream(filePath, { flags: 'a' })
    debug(`File reopened: ${filePath}`)
  }

  req.on('data', (data) => {
    debug(`bytes received: ${upload.bytesReceived}`)
    upload.bytesReceived += data.length
  })

  req.pipe(fileStream)

  fileStream.on('close', () => {
    if (upload.bytesReceived === req.headers['x-file-size']) {
      debug('Upload finished')
      delete uploads[fileId]
      res.end(`Success ${upload.bytesReceived}`)
    } else {
      debug(`File unfinished, stopped at ${upload.bytesReceived}`)
      res.end()
    }
  })

  fileStream.on('error', (err) => {
    debug('fileStream error')
    res.writeHead(500, 'File error')
    res.end()
  })
}

function onStatus(req, res) {
  let fileId = req.headers['x-file-id']
  let upload = uploads[fileId]
  debug(`onStatus fileId: ${fileId} upload: ${upload}`)

  if (!upload) {
    res.end('0')
  } else {
    res.end(String(upload.bytesReceived))
  }
}

function accept(req, res) {
  console.log(req.url, path.extname(req.url))
  if (req.url === '/status') {
    onStatus(req, res)
  } else if (req.url === '/upload' && req.method === 'POST') {
    onUpload(req, res)
  } else {
    fileServer.serve(req, res)
  }
}

if (!module.parent) {
  http.createServer(accept).listen(8080)
  console.log('Server listening at port 8080')
} else {
  exports.accept = accept
}

```



```javascript
// uploader.js
class Uploader {
  constructor({ file, onProgress }) {
    this.file = file
    this.onProgress = onProgress

    // 创建唯一标识文件的 fileId
    // 我们还可以添加用户会话标识符（如果有的话），以使其更具唯一性
    this.fileId = file.name + '-' + file.size + '-' + +file.lastModifiedDate
  }

  async getUploadedBytes() {
    let response = await fetch('/status', {
      headers: {
        'X-File-Id': this.fileId
      }
    })

    if (response.status != 200) {
      throw new Error("Can't get uploaded bytes: " + response.statusText)
    }

    let text = await response.text()

    return +text
  }

  async upload() {
    this.startByte = await this.getUploadedBytes()

    let xhr = (this.xhr = new XMLHttpRequest())
    xhr.open('POST', 'upload', true)

    // 发送文件 id，以便服务器知道要恢复哪个文件
    xhr.setRequestHeader('X-File-Id', this.fileId)
    // 发送我们要从哪个字节开始恢复，因此服务器知道我们正在恢复
    xhr.setRequestHeader('X-Start-Byte', this.startByte)

    xhr.upload.onprogress = (e) => {
      this.onProgress(this.startByte + e.loaded, this.startByte + e.total)
    }

    console.log('send the file, starting from', this.startByte)
    xhr.send(this.file.slice(this.startByte))

    // return
    //   true —— 如果上传成功，
    //   false —— 如果被中止
    // 出现 error 时将其抛出
    return await new Promise((resolve, reject) => {
      xhr.onload = xhr.onerror = () => {
        console.log(
          'upload end status:' + xhr.status + ' text:' + xhr.statusText
        )

        if (xhr.status == 200) {
          resolve(true)
        } else {
          reject(new Error('Upload failed: ' + xhr.statusText))
        }
      }

      // onabort 仅在 xhr.abort() 被调用时触发
      xhr.onabort = () => resolve(false)
    })
  }

  stop() {
    if (this.xhr) {
      this.xhr.abort()
    }
  }
}
```



```html
// upload.html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <script src="uploader.js"></script>

    <form
      name="upload"
      method="POST"
      enctype="multipart/form-data"
      action="/upload"
    >
      <input type="file" name="myfile" />
      <input
        type="submit"
        name="submit"
        value="Upload (Resumes automatically)"
      />
    </form>

    <button onclick="uploader.stop()">Stop upload</button>

    <div id="log">Progress indication</div>

    <script>
      function log(html) {
        document.getElementById('log').innerHTML = html
        console.log(html)
      }

      function onProgress(loaded, total) {
        log('progress ' + loaded + ' / ' + total)
      }

      let uploader

      document.forms.upload.onsubmit = async function (e) {
        e.preventDefault()

        let file = this.elements.myfile.files[0]
        if (!file) return

        uploader = new Uploader({ file, onProgress })

        try {
          let uploaded = await uploader.upload()

          if (uploaded) {
            log('success')
          } else {
            log('stopped')
          }
        } catch (err) {
          console.error(err)
          log('error')
        }
      }
    </script>
  </body>
</html>
```

