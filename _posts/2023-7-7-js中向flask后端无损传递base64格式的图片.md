---
redirect_from: /_posts/2017-12-13-NVCPC-preview-contest-from-Tailmon/
title: js中向flask后端无损传递base64格式的图片
tags:
  - 网页开发
---

### 当使用 JavaScript 向 Flask 后端传递图像时，可以选择使用 Base64 格式进行无损传递。Base64 是一种将二进制数据编码为 ASCII 字符串的方法，它可以直接在文本中传递图像数据。

# 使用FileReader
```
// 选择图像文件
const fileInput = document.getElementById('file-input');
fileInput.addEventListener('change', () => {
  const file = fileInput.files[0];

  // 创建 FileReader 对象
  const reader = new FileReader();

  // 读取文件内容
  reader.onload = () => {
    const base64Img = reader.result;

    // 发送到后端
    sendToBackend(base64Img);
  };

  // 将文件内容读取为 DataURL（Base64）
  reader.readAsDataURL(file);
});

// 向后端发送 Base64 图像数据
function sendToBackend(base64Img) {
  fetch('/upload', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ image: base64Img })
  })
  .then(response => {
    // 处理响应
  })
  .catch(error => {
    // 处理错误
  });
}
```
### 上述代码监听了一个文件输入框的变化事件，当用户选择图像文件时，会将文件内容读取为 Base64 格式，并通过 sendToBackend 函数将数据发送到后端。
### 在 Flask 后端接收到数据后，可以使用以下代码提取 Base64 图像数据并保存为图像文件
```
from flask import Flask, request
import base64

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload():
    data = request.get_json()
    base64Img = data['image']

    # 提取图像数据
    _, img_data = base64Img.split(',')

    # 保存为图像文件
    with open('image.png', 'wb') as f:
        f.write(base64.b64decode(img_data))

    return 'Image uploaded successfully'

if __name__ == '__main__':
    app.run()
```
# 使用canvas

### 实测该方法将图片传到flask后端时有时图片会出现相较于原图变模糊的情况，本人尚不清楚原因
```
// 选择图像文件
const fileInput = document.getElementById('file-input');
fileInput.addEventListener('change', () => {
  const file = fileInput.files[0];

  // 创建图像对象
  const img = new Image();

  // 图像加载完成后执行回调函数
  img.onload = () => {
    // 创建 Canvas 元素
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');

    // 设置 Canvas 大小与图像大小一致
    canvas.width = img.width;
    canvas.height = img.height;

    // 在 Canvas 上绘制图像
    ctx.drawImage(img, 0, 0);

    // 将 Canvas 数据转换为 Base64 字符串
    const base64Img = canvas.toDataURL('image/png');

    // 发送到后端
    sendToBackend(base64Img);
  };

  // 将文件 URL 设置为图像源
  img.src = URL.createObjectURL(file);
});

// 向后端发送 Base64 图像数据（与前一个示例中的代码相同）
function sendToBackend(base64Img) {
  // ...
}
```
### 上述代码与前一个示例中的代码类似，不过在将图像内容转换为 Base64 格式之前，我们先使用 Canvas 绘制了图像。通过创建一个 Canvas 元素，并设置其宽度和高度与图像大小一致，在 Canvas 上调用 drawImage 方法绘制图像，然后使用 toDataURL 方法将 Canvas 数据转换为 Base64 字符串。
### 在 Flask 后端的代码不需要做任何改变，可以直接使用前一个示例中的代码来处理接收到的 Base64 图像数据。

## 无论是使用 FileReader 还是 Canvas，都可以实现将图像转换为 Base64 格式并传递给 Flask 后端。具体使用哪种方法取决于你的需求和个人喜好。希望对你有所帮助！
