---
title: flask-login
date: 2018-09-30 17:36:59
categories: web
tags: flask
---
flas-login
<!--more-->

# 0x00 原理

## 首次登陆
![image](E2AA8D0D2E8E4572AD35D86D511E0E95)

## 第二次登陆
![image](AD3CBFB849794F18BFDF9978B137F0EB)

# 0x01 实现

## SQLAlchemy操作数据库

首先给定数据库信息

### 结构
传统的数据库表是一个二维表。而SQLAlchemy使用对象来表示。
```python
# 传统的
[
    ('1', 'Michael'),
    ('2', 'Bob'),
    ('3', 'Adam')
]
# SQLAlchemy
class User(object):
    def __init__(self, id, name):
        self.id = id
        self.name = name

[
    User('1', 'Michael'),
    User('2', 'Bob'),
    User('3', 'Adam')
]

```
### 用法
#### 直接用法 
[直接](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014320114981139589ac5f02944601ae22834e9c521415000)
#### flask中的用法

```python
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
# 引擎 数据库://username:password@ip/database
app.config['SQLALCHEMY_DATABASE_URI'] = "mysql://root:root@127.0.0.1/taskt"
# 创建对象
db = SQLAlchemy(app)
# 定义用户模型
class User(UserMixin, db.Model):
    __tablename__ = 'user'
    userid = db.Column('userid', Integer, primary_key=True)
    username = db.Column('username', String(30), unique=True)
    email = db.Column('email', String(50), unique=True)
    password = db.Column('password', String(100), unique=False)

    def __init__(self, username, password, email=None):
        self.username = username
        self.password = password
        self.email = email

        def is_authenticated(self):
        return True

# 获取数据
username = form.username.data
password = form.password.data
email = form.email.data
# 添加数据
db.session.add(User(username=username, password=password, email=email))
db.session.commit()
# 查询数据
db_pass_user = User.query.filter_by(username=username).first()
```
## flask-login
```python
#!/usr/bin/env python

# encoding: utf-8
from flask import Flask, Blueprint, render_template, request, redirect
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import Integer, String

from flask_login import (LoginManager, login_required, login_user,

                         logout_user, UserMixin, )
from wtforms import StringField, PasswordField, SubmitField, Form
from flask_cors import CORS

app = Flask(__name__)
CORS(app, supports_credentials=True)
app.config['SQLALCHEMY_DATABASE_URI'] = "mysql://root:root@127.0.0.1/taskt"
db = SQLAlchemy(app)


class LoginForm(Form):
    username = StringField('username')
    password = PasswordField('password')
    email = StringField('email')
    submit = SubmitField('submit')


# user models

class User(UserMixin, db.Model):
    __tablename__ = 'user'
    userid = db.Column('userid', Integer, primary_key=True)
    username = db.Column('username', String(30), unique=True)
    email = db.Column('email', String(50), unique=True)
    password = db.Column('password', String(100), unique=False)

    def __init__(self, username, password, email=None):
        self.username = username
        self.password = password
        self.email = email

    def is_authenticated(self):
        return True

    def is_actice(self):
        return True

    def is_anonymous(self):
        return False

    def get_id(self):
        return str(self.userid)  # User.query.filter_by(username=self.username).first().userid

    def __repr__(self):
        return '<%s %s>' % (self.username, self.password)


# flask-login
app.secret_key = 's3cr3t'
login_manager = LoginManager()
login_manager.session_protection = 'strong'
login_manager.login_view = 'login'
login_manager.init_app(app)


@login_manager.user_loader
def load_user(user_id):
    print("user_id: " + user_id)
    return User.query.filter_by(userid=user_id).first()


@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm(request.form)
    if request.method == 'POST' and form.validate():
        username = form.email.data
        password = form.password.data
        db_pass_user = User.query.filter_by(username=username).first()

        if db_pass_user and db_pass_user.password == password:

            login_user(db_pass_user)
            return render_template("home.html", data=db_pass_user)
        else:
            return "wrong username/password "
    else:
        return render_template("login.html", form=form)


@app.route("/register", methods=['POST', 'GET'])
def register():
    form = LoginForm(request.form)
    if request.method == 'POST' and form.validate():
        username = form.username.data
        password = form.password.data
        email = form.email.data
        db.session.add(User(username=username, password=password, email=email))
        db.session.commit()
        return "success"


@app.route("/")
@app.route("/index")
def index():
    return render_template("/index.html", form=LoginForm())  # redirect('/login')


@app.route('/logout', methods=['GET', 'POST'])
@login_required
def logout():
    logout_user()

    return "logout page"


# test method

@app.route('/test')
@login_required
def test():
    return "yes , you are allowed"


app.run(debug=True)
```

# 0x02  跨域问题
```python
from flask_cors import CORS
CORS(app, supports_credentials=True)
```