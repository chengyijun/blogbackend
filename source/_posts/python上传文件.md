---
title: python上传文件
date: 2022-09-09 13:42:28
tags:
---

```python
from pathlib import Path
from fastapi import FastAPI, UploadFile, File
import uvicorn

app = FastAPI()


@app.post("/upload")
def test1(file: UploadFile = File()):
    # 这里的参数file必须与构建的file对象的key一致才行
    print(file.filename)
    with open(f"upload-{file.filename}", "wb") as f:
        f.write(file.file.read())
    return {"mes": "ok"}


if __name__ == '__main__':
    uvicorn.run(app=f"{Path(__file__).name.split('.')[0]}:app",
                host="localhost",
                port=3000,
                debug=True,
                reload=True,
                workers=1)
```

```python
import requests

# 要上传的文件
files = {'file': ('1.jpg', open('1.jpg', 'rb'), "image/jpg")}
url = "http://localhost:3000/upload"
r = requests.post(url, files=files)
print(r.text)
```

