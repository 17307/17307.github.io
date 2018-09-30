---
title: flask-token认证
date: 2018-09-30 17:39:06
categories: web
tags: flask
---
flask 前后端分离 token认证
<!--more-->


# 0x00 用curl提交数据

## 坑

`curl -H "Accept: application/json" -H "Content-type:application/json" -X POST -d "{\"username\": \"123\", \"password\": \"234\"}" http://127.0.0.1:5000/api/test `  
不知道为什么自己的单引号写法不可以。  
网上大部分人给的都是 用单引号扩起json内容，里面的双引号不转义。即
`curl -H "Accept: application/json" -H "Content-type:application/json" -X POST -d '{"username": "123", "password": "234"}' http://127.0.0.1:5000/api/test`  但这个测试错误。

## curl用法
```bash
# 提交cookie
curl http://man.linuxde.net --cookie "user=root;pass=123456"
# user-agent
curl URL --user-agent "Mozilla/5.0"
curl URL -A "Mozilla/5.0"
# 其他信息头
curl -H "Host:man.linuxde.net" -H "accept-language:zh-cn" URL
# HTTP基础认证
curl -u user:pwd http://man.linuxde.net
# `-i`参数可以显示http response的头信息，连同网页代码一起。
curl -i www.sina.com
# GET
curl example.com/form.cgi?data=xxx
# POST
curl -X POST --data "data=xxx" example.com/form.cgi
# cookie
curl --cookie "name=xxx" www.example.com
```

# 0x01 与flask-login的不同
## flask-login
在flask-login中，对于需要登陆的函数，修饰符为`@login_required
`。它的回调方法是：`@login_manager.user_loader`
```python
@login_manager.user_loader
def load_user(user_id):
    print("user_id: " + user_id)
    return User.query.filter_by(userid=user_id).first()

@app.route('/test')
@login_required
def test():
    return "yes , you are allowed"

'''
在load_user方法中，会自动在session中得到用户的user_id。然后判断是否有这个user_id的用户。
'''    
```  
## 前后分离-HTTPBasicAuth
在前后端分离的token验证过程中，修饰符为`@auth.login_required`,它的回调方法是 `@auth.verify_password`.
```python
@auth.verify_password
def verify_password(username_or_token, password):
    if request.path == "/api/login":
        # 支持用表单的方式提交数据
        username_and_password_post = request.get_json()
        if username_and_password_post.get('email') is not None:
            username_or_token = username_and_password_post['email']
        if username_and_password_post.get('password') is not None:
            password = username_and_password_post['password']

        user = User.query.filter_by(username=username_or_token).first()
        if not user or not user.verify_password(password):
            return False
    else:
        user = User.verify_auth_token(username_or_token)
        if not user:
            return False
    g.user = user
    return True
    
@app.route("/", methods=['POST', 'GET'])
@auth.login_required
def index():
    return jsonify('Hello, %s' % g.user.username)
    
'''
被@auth.login_required拦截后，会调用verify_password方法。
两个参数会自动赋值为HTTP基础认证中的username和password。
'''
```
## 总结
实际上，**flask-login**中的`@login_manager.user_loader`是会话管理功能。在session中取值。  
而 **HTTPBasicAuth** 中的`@auth.verify_password`做的更多的是`token`或者**其他形式**的认证。


# 0x02 代码
```python
"""
api.py
"""
# -*- coding:utf-8 -*-

from passlib.apps import custom_app_context
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer, SignatureExpired, BadSignature
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_httpauth import HTTPBasicAuth
from flask_cors import CORS
from flask import jsonify, request, abort, g

app = Flask(__name__)
# flask的跨域解决
CORS(app, supports_credentials=True)
# 可以获取config.py的内容
app.config.from_object('config')
db = SQLAlchemy(app)
auth = HTTPBasicAuth()


class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(32), index=True)
    password = db.Column(db.String(128))

    # 密码加密
    def hash_password(self, password):
        self.password = custom_app_context.encrypt(password)

    # 密码解析
    def verify_password(self, password):
        return custom_app_context.verify(password, self.password)

    # 获取token，有效时间10min
    def generate_auth_token(self, expiration=600):
        s = Serializer(app.config['SECRET_KEY'], expires_in=expiration)
        return s.dumps({'id': self.id})

    # 解析token，确认登录的用户身份
    @staticmethod
    def verify_auth_token(token):
        s = Serializer(app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except SignatureExpired:
            return None  # valid   token, but expired
        except BadSignature:
            return None  # invalid token
        user = User.query.get(data['id'])
        return user


# db.create_all() 创建表

@app.route("/", methods=['POST', 'GET'])
@auth.login_required
def index():
    return jsonify('Hello, %s' % g.user.username)


@app.route('/api/register', methods=['POST'])
def new_user():
    json_data = request.get_json()
    username = json_data['username']
    password = json_data['password']
    if username is None or password is None:
        abort(400)  # missing arguments
    if User.query.filter_by(username=username).first() is not None:
        abort(400)  # existing user
    user = User(username=username)
    user.hash_password(password)
    db.session.add(user)
    db.session.commit()
    return jsonify({'username': user.username})


@auth.verify_password
def verify_password(username_or_token, password):
    if request.path == "/api/login":
        username_and_password_post = request.get_json()
        if username_and_password_post.get('email') is not None:
            username_or_token = username_and_password_post['email']
        if username_and_password_post.get('password') is not None:
            password = username_and_password_post['password']

        user = User.query.filter_by(username=username_or_token).first()
        if not user or not user.verify_password(password):
            return False
    else:
        user = User.verify_auth_token(username_or_token)
        if not user:
            return False
    g.user = user
    return True


@app.route('/api/login', methods=['POST', 'GET'])
@auth.login_required
def get_auth_token():
    token = g.user.generate_auth_token()
    token = str(token, encoding='utf8')
    return jsonify(token)


@app.route('/api/test', methods=['POST'])
def test():
    b = request.get_json()
    print(b)
    return "test"


app.run(debug=True)





"""
config.py
"""
import os
basedir = os.path.abspath(os.path.dirname(__file__))

SQLALCHEMY_DATABASE_URI = "mysql://root:root@127.0.0.1/flaskrest"
SQLALCHEMY_MIGRATE_REPO = os.path.join(basedir, 'db_repository')
SQLALCHEMY_TRACK_MODIFICATIONS = True
BASEDIR = basedir
# 安全配置
CSRF_ENABLED = True
SECRET_KEY = 'jssssaqer123dsaf/sdf\sdf'
```
