## Flask-HTTPAuth

### 安装和启用
```bash
pip install Flask-HTTPAuth
```
> 创建扩展对象实例：
```python
from flask import Flask
from flask_httpauth import HTTPBasicAuth

app = Flask(__name__)
auth = HTTPBasicAuth()
```

> 通过添加 login_required 装饰器可以要求相应的路由必须进行认证:
```python
@app.route('/api/resource')
@auth.login_required
def get_resource():
    return jsonify({'data': 'hello world'})
```
> Flask-HTTPAuth提供了几种不同的认证方法，比如 HTTPBasicAuth，HTTPTokenAuth，MultiAuth 和 HTTPDigestAuth。创建实例中我们使用的是 HTTPBasicAuth。

### 基于密码的认证

> Flask-HTTPAuth 中的 verify_password 回调函数将会根据提供的 username 和 password 的组合的，返回 True(通过验证) 或者 False(未通过验证)
```python
from flask_httpauth import HTTPBasicAuth

auth = HTTPBasicAuth()


@auth.verify_password
def verify_password(username, password):
    user = User.query.filter_by(username=username).first()
    if not user or not user.verify_password(password):
        return False
    g.user = user
    return True
```
> 这个函数将会根据 username 找到用户，并且使用 verify_password() 方法验证密码。如果认证通过的话，用户对象将会被存储在 Flask 的 g 对象中，这样视图就能使用它。

使用 Postman 请求 api 的两种传递 username，password 方式

> 1. Authorization
> 2. HTTP 头信息

Postman 中的认证方式存在很多种，此处我们根据代码中使用的 HTTPBasicAuth 选择 basic auth

![Alt text](auth.png)

当在 Postman 中以原文形式将 username，password 存储在 HTTP 头部信息中传递，后端拿不到对应的值。众所周知 HTTP 的头部信息都是键值对的形式，HTTPBasicAuth 规定的也是键值对的形式，不过稍微有所不同。
```bash
Authorization: basic base64(account:password)
```
键值对的键为 Authorization，值的开头是 basic + 空格 + base64加密过后的账号和密码。
> (账后和密码中间有冒号)

![Alt text](postman.png)

当然，除了 Postman，我们也可以使用 Curl 命令请求对应的 api
```shell
curl -u hello:world -i -X GET http://127.0.0.1:5000/api/resource
```

### 基于令牌的认证
在对 HTTP 形式的 api 发请求时，大部分情况我们不是通过用户名密码做验证，而是通过一个令牌，也就是 Token 来做验证。此时，我们就使用 Flask-HTTPAuth 中的 HTTPTokenAuth

同 HTTPBasicAuth 类似，它也提供 login_required 装饰器来认证视图函数。不同的是，它没有 verify_password 装饰器，而是利用 verify_token 验证令牌

```python
from flask import current_app, g
from flask_httpauth import HTTPTokenAuth
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer, BadSignature, SignatureExpired
from app.models import User

auth = HTTPTokenAuth(scheme='Token')


# 生成 Token
def generate_auth_token(uid, expiration=7200):
    s = Serializer(current_app.config['SECRET_KEY'], expires_in=expiration)
    return s.dumps({
        'uid': uid,
    })


# 验证 Token
@auth.verify_token
def verify_token(token):
    s = Serializer(current_app.config['SECRET_KEY'])
    try:
        data = s.loads(token)
    except BadSignature:
        raise AuthFailed(msg='token is invalid')
    except SignatureExpired:
        raise AuthFailed(msg='token is expired')
    g.user = User.query.get_or_404(data['uid'])
    return True
```
初始化 HTTPTokenAuth 时，我们传入了 scheme=’Token'。这个 scheme 就是我们在发送请求时，在 HTTP 头 Authorization 中要用的 scheme 字段。其中 scheme 默认值为 Bearer

> 使用 Postman 请求 api

![Alt text](token_auth.png)

> 使用 Curl 命令请求 api
```shell
curl -X GET -H "Authorization: Token eyJ1c2VybmFtZSI6ImpvaG4ifQ.80IN9uAm395ywtEfKTdXINCZx0oi1M3BjA11xU-gtei-mzOE360KIgHox_JYtNR2BqzqH5tJPi2jtJlXtbf-eQ" http://localhost:5000/api/resource
```
