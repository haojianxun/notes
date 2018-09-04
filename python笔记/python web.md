# python web

```python
def application(environ, start_response):
  start_response('200 OK', [('Content-Type', 'text/html')])
  return '<h1>Hello, web!</h1>'

if __name__ =='__main__':
  from wsgiref.simple.server import make_server
  server = make.server('' , 3000 , application)
  try:
    server.server_forever()
  except KeyboardInterrupt:
    server.shutdown()
  
```

一个容器：gunicorn  可以用pip安装