# Getting Started

确保你已经在开始之前已经安装了 [pip](https://pip.pypa.io/en/stable/installing/)和最少是版本3.5的python
. Sanic 使用了新的语法 `async`/`await`, 所以早期的python版本是不会工作的.

1. 安装 Sanic: `python3 -m pip install sanic`
2. 生成带有以下代码内容的 `main.py`:

  ```python
  from sanic import Sanic
  from sanic.response import json

  app = Sanic()

  @app.route("/")
  async def test(request):
      return json({"hello": "world"})

  if __name__ == "__main__":
      app.run(host="0.0.0.0", port=8000)
  ```
  
3. 运行: `python3 main.py`
4. 在浏览器里面打开这个地址 `http://0.0.0.0:8000`. 你应该会看到这么一个页面 *Hello world!*.

现在你已经有一个正在运行的Sanic服务器了!
