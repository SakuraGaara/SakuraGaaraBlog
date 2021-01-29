---
title: Flask笔记-SQLAlchemy应用
categories: wtforms
tags:
  - wtforms
abbrlink: 54760
date: 2020-01-02 00:00:00
---

通常情况下，一个简单的注册试图
```python
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()

class User(db.Model):
    id = Column(Integer, primary_key=True)
    nickname = Column(String(24), nullable=False)
    email = Column(String(50), unique=True, nullable=False)
    password = Column('password', String(128), nullable=False)
    status = Column(SmallInteger, default=1)

@web.route('/register',methods=['GET','POST'])
def register():
	if request.method == 'POST':
		user=User()
		user.nickname = request.args['nickname']
		user.email = request.args['email']
        user.password = request.args['password']
		db.session.add(user)
		db.session.commit()
		redirect(url_for('web.login'))
    return render_template('auth/register.html')
```
<!--more-->

### 参数验证
但是仅仅只是这样简单的接收任意参数，是非常不安全的，可以定义一个验证表单的类，来接受参数

```python
from flask_sqlalchemy import SQLAlchemy
from wtforms import Form,StringField, IntegerField, PasswordField
from wtforms.validators import DataRequired,Email,NumberRange,Length,ValidationError

class RegisterForm(Form):
    email = StringField(validators=[DataRequired(),
                                    Length(8, 64),
                                    Email(message='电子邮箱不符合规范')])

    password = PasswordField(validators=[
        DataRequired(message='密码不可以为空，请输入正确的密码'),
        Length(6,32)])

    nickname = StringField(validators=[
        DataRequired(), Length(2,16, message='昵称至少需要两个字符，最多10个字符')
    ])
		

@web.route('/register',methods=['GET','POST'])
def register():
    form = RegisterForm(request.form)
    if request.method == 'POST' and form.validate():
        user = User()
        user.nickname = form.nickname.data
        user.email = form.email.data
        user.password = form.password.data
        db.session.add(user)
        db.session.commit()
        redirect(url_for('web.login'))
    return render_template('auth/register.html', form=form)
```

### 密码加密策略
但是这样存在隐患，通常密码并非明文存储，在werkzeug中，提供了加密和校验方法``generate_password_hash  check_password_hash``,将password定义为方法在转换成属性

改写User类
```python
from werkzeug.security import generate_password_hash,check_password_hash

class User(db.Model):
    id = Column(Integer, primary_key=True)
    nickname = Column(String(24), nullable=False)
    email = Column(String(50), unique=True, nullable=False)
    _password = Column('password', String(128), nullable=False)
    status = Column(SmallInteger, default=1)

    @property
    def password(self):
    	return self._password
    

    @password.setter
    def password(self, raw):
        self._password=generate_password_hash(raw)

    def check_password(self, raw):
    	"""
    	return: True or False
    	"""
        return check_password_hash(self._password, raw)
```

### 基础类继承
通常情况，User定义数据库类，在数据库角度看来，一个类就是一个表，每个表都有一些状态，创建时间等基础的数据，那么我们可以把它单独抽出，定义一个基础类
```python
class Base(db.Model):
    __abstract__ = True
    # create_time = Column()
    status = Column(SmallInteger, default=1)

class User(Base):
    id = Column(Integer, primary_key=True)
    nickname = Column(String(24), nullable=False)
    email = Column(String(50), unique=True, nullable=False)
    _password = Column('password', String(128), nullable=False)

    @property
    def password(self):
    	return self._password
    

    @password.setter
    def password(self, raw):
        self._password=generate_password_hash(raw)

    def check_password(self, raw):
        return check_password_hash(self._password, raw)
```


### 简化视图函数参数获取
在试图函数中，有很多的参数需要获取``user.nickname = form.nickname.data``这样的类似操作，太多的获取会让视图函数看起来非常的臃肿，看起来操作类似，似乎又有逻辑可循
那么，我们有一些特殊的方法，是可以简化操作
```python
class Base(db.Model):
    __abstract__ = True
    # create_time = Column()
    status = Column(SmallInteger, default=1)

    def set_attrs(self, attr_dict):
        for key, value in attr_dict.items():
            if hasattr(self, key) and key != 'id':
                setattr(self, key, value)


@web.route('/register',methods=['GET','POST'])
def register():
    form = RegisterForm(request.form)
    if request.method == 'POST' and form.validate():
    	try:
	        user = User()
	        user.setattr(form.data)
	        db.session.add(user)
	        db.session.commit()
	    except Exception as e:
            self.session.rollback()
            raise e
        redirect(url_for('web.login'))
    return render_template('auth/register.html', form=form)
```


### 上下文管理器应用
在每次提交数据，都会有``try except``操作，当有大量的试图函数时，这样看起来依然感觉试图函数臃肿
```python
from flask_sqlalchemy import SQLAlchemy as _SQLAlchamy
from contextlib import contextmanager

class SQLAlchemy(_SQLAlchamy):
	@contextmanager
	def auth_commit(self):
		try:
			yield
			self.session.commit()
		except Exception as e:
			self.session.rollback()
			raise e
		
db = SQLAlchemy()


@web.route('/register',methods=['GET','POST'])
def register():
    form = RegisterForm(request.form)
    if request.method == 'POST' and form.validate():
    	while db.auth_commit():
	        user = User()
	        user.setattr(form.data)
	        db.session.add(user)
        redirect(url_for('web.login'))
    return render_template('auth/register.html', form=form)
```

# jinja2模板渲染
```python
trade_gifts = Gift.query.filter_by(isbn=isbn, launched=False).all()
trade_gifts_model = TradeInfo(trade_gifts)

class TradeInfo:
    def __init__(self, goods):
        self.total = 0
        self.trades = []
        self.__parse(goods)

    def __parse(self, goods):
        self.total = len(goods)
        self.trades = [self.__map_to_trade(single) for single in goods]

    @staticmethod
    def __map_to_trade(single):
        time = single.create_datetime.strftime("%Y-%m-%d") if single.create_datetime else None
        return dict(
            user_name=single.user.nickname,
            time=time,
            id=single.id
        )
```