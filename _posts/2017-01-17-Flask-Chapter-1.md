---
title: Flask Chapter 1
---

Writing Restful Api's in Flask
===================


In this series of 10 Chapters we will learn basics of writing a **restful api**[^rest api] service using flask and create a seed app which can be used as building block for large web applications.  This series target audience with basic knowledge of using Flask or any other MVC framework.

----------


Chapter 1
-------------

### Starting Flask Server

``` python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

Save a file with the above code and start the server by running following commands.

``` bash
$ export FLASK_APP=<filename>
$ flask run
```

###  Adding Database Model's


> **Note:**

> - Install SQLAlchemy and Flask-SQLAlchemy.
> - You must have sqlite installed and running.

We will be adding three models User, Role, UserRole.

``` python
import os
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///{}'.format(os.path.join(basedir, 'test.db'))
db = SQLAlchemy(app)


class User(db.Model):
	__tablename__ = 'user'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80))
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())

    def __repr__(self):
        return '<id {} user {}>'.format(self.id, self.name)


class Role(db.Model):
	__tablename__ = 'role'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=True)
    description = db.Column(db.Text, unique=True)
	created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())

	def __repr__(self):
        return '<id {} role {}>'.format(self.id, self.name)


class UserRole(db.Model):
	__tablename__ = 'user_role'

	id = db.Column(db.Integer, primary_key=True)
	user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
	role_id = db.Column(db.Integer, db.ForeignKey('role.id'))
	created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())

	def __repr__(self):
        return '<user_id {} role_id {}>'.format(self.user_id, self.role_id)


@app.route('/')
def hello_world():
    return 'Hello, World!'
```
The user model and role models are self explanatory. *UserRole* model might look a little confusing to some,
UserRole model is called an association table and it acts like a junction between **User** and **Role**. It stores the data of which user is connected to which role.

>**Example:**

|  id      |   user_id   |  role_id  |
|:--------:|:-----------:|:---------:|
|  1       |    1        |  1        |
|  2       |    1        |  2        |
|  3       |   2         |  1        |

>This shows that user with id 1 has two roles with id 1 and 2 associated with it and user with id 2 has one role with id 1 associated with it.

Every model contains a **`id`** field which is used as a primary key and a **`created_on`** and **`updated_on`** which keeps track of when the row was created and when was it last time updated.
All the models also contains a **`__repr__()`** function which makes the objects more representable and alse help will debugging.

>**Note**:

>Printing an object of user model in console (`print(User())`) will print something like this `<id None name None>`
>Lets add some data and see.
>`>> user = User()`
>`>> user.name = 'username'`
>`>> user.id = 1`
>After adding some data it will look like this.
>`>> print(user)`
> `>> <id 1 user username>`

Lets see how we can make it more readable and concise by writting two more classes BaseMixin and ReprMixin.


```python
import re
from sqlalchemy.ext.declarative import declared_attr
from sqlalchemy.schema import Index

def to_underscore(name):

    s1 = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
    return re.sub('([a-z0-9])([A-Z])', r'\1_\2', s1).lower()


 class BaseMixin(object):

     @declared_attr
     def __tablename__(self):
         return to_underscore(self.__name__)

     id = db.Column(db.Integer, primary_key=True)
     created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
     updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())


class ReprMixin(object):

    __repr_fields__ = ['id', 'name']

    def __repr__(self):
        fields = {f: getattr(self, f, '<BLANK>') for f in self.__repr_fields__}
        pattern = ['{0}={{{0}}}'.format(f) for f in self.__repr_fields__]
        pattern = ' '.join(pattern)
        pattern = pattern.format(**fields)
        return '<{} {}>'.format(self.__class__.__name__, pattern)
```

By adding BaseMixin and ReprMixin we can put all the repeated code under one class and inherit those classes where ever required.
Our BaseMixin class contains three attributes **`id, created_on, updated_on`** which were common among all the models. It also contains a declarative attribute which automatically generates the table name using the class name, making code more readable, concise and less error prone.

Class ReprMixin helps us by adding a generic function which uses **`id`** and **`name`** by default attributes of class to represent the objects. The **`__repr__fields `** attribute can easily be overridden in sub classes as required.

Our new code looks like this

``` python
import os
import re
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.ext.declarative import declared_attr

basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///{}'.format(os.path.join(basedir, 'test.db'))
db = SQLAlchemy(app)
db.create_all()


def to_underscore(name):

    s1 = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
    return re.sub('([a-z0-9])([A-Z])', r'\1_\2', s1).lower()


class BaseMixin(object):

    @declared_attr
    def __tablename__(self):
        return to_underscore(self.__name__)

    id = db.Column(db.Integer, primary_key=True)
    created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())


class ReprMixin(object):

    __repr_fields__ = ['id', 'name']

    def __repr__(self):
        fields = {f: getattr(self, f, '<BLANK>') for f in self.__repr_fields__}
        pattern = ['{0}={{{0}}}'.format(f) for f in self.__repr_fields__]
        pattern = ' '.join(pattern)
        pattern = pattern.format(**fields)
        return '<{} {}>'.format(self.__class__.__name__, pattern)


class User(db.Model, BaseMixin, ReprMixin):
    __tablename__ = 'user'

    name = db.Column(db.String(80))
    email = db.Column(db.String(120), unique=True)

class Role(db.Model, BaseMixin, ReprMixin):
    __tablename__ = 'role'

    name = db.Column(db.String(80), unique=True)
    description = db.Column(db.Text, unique=True)


class UserRole(db.Model, BaseMixin, ReprMixin):
    __tablename__ = 'user_role'
    __repr_fields__ = ['user_id', 'role_id']

    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    role_id = db.Column(db.Integer, db.ForeignKey('role.id'))


@app.route('/')
def hello_world():
    return 'Hello, World!'

```

Start a python  console and create your first object.

``` bash
$ python
```
``` python
>> from app import *
>> user = User('saurabh', 'saurabh.1e1@gmail.com')
>> db.session.add(user)
>> db.session.commit()
>> user
>> <User id=1 name=saurabh>

```

### Adding More Fields

Lets complete the our models. Our goal is to create a database to hold all the necessary data of a user, all the available roles and info of different roles associated with the users.

#### **Updated User Model:**

```  python

from sqlalchemy.ext.hybrid import hybrid_property

class User(db.Model, BaseMixin, ReprMixin):
    __tablename__ = 'user'

	email = db.Column(db.String(120), unique=True)
	password = db.Column(db.String(255))
	first_name = db.Column(db.String(40), nullable=False)
	last_name = db.Column(db.String(40))
	profile_picture = db.Column(db.Text()))
	bio = db.Column(db.Text()))
	active = db.Column(db.Boolean())
	last_login_at = db.Column(db.DateTime())
	date_of_birth = db.Column(db.Date)

	@hybrid_property
	def name(self):
		return '{}'.format(self.first_name) + (' {}'.format(self.last_name) if self.last_name else '')

```

>**Note:**
> - We have added two new columns **`first_name, last_name`** instead of name and introduced a hybrid property.
> - Now we can use name as a attribute of user class instance, like **`user.name`**.




### Adding Relations

**So what are relations?**
Database tables are often related to one another, SqlAlchemy relations makes managing and working with these relationships easy. Defining relations powerful method chaining and querying capabilities.

To know more about what sqlalchemy relationships are read [this](http://www.ergo.io/blog/sqlalchemy-relationships-from-beginner-to-advanced/) and [this](http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html).

**User Model**
``` python
class User(db.Model, BaseMixin, ReprMixin):
	roles = db.relationship('Role', secondary='user_role', back_populates='users')
```
**Role Model**

``` python
  class Role(db.Model, BaseMixin, ReprMixin):
	 users = db.relationship('User', secondary='user_role', back_populates='roles')
```
**UserRole Model**

``` python
class UserRole(db.Model, BaseMixin, ReprMixin):
	user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
	role_id = db.Column(db.Integer, db.ForeignKey('role.id'))
	role = db.relationship('Role', foreign_keys=[role_id])
	user = db.relationship('User', foreign_keys=[user_id])
```

We have added a relation **`roles`** in user model and **`users`** in role model, this now allows us to access, add and remove and update a user's roles from a user object and vice a versa.

>**Example:**

**Adding an user and a role**.

`>> user = User()`

` >> user.first_name = 'saurabh'`

` >> user.email = 'example@gmail.com'`

` >> db.session..add(user)`

` >> db.session.commit()`

` >> role = Role()`

` >> role.name = 'admin'`

` >> db.session..add(role)`

` >> db.session.commit()`


**Now we can do this**

`>> user.roles.append(role)`

`>> db.session.commit()`

`>> print(user.roles)`

`>> [<Role id=1 name=admin>]`

`>> print(role.users)`

` >> [<User id=1 name=saurabh>]`



Our UserRole model also contains two relationships which are acting as junction between roles relation in user and users relationship in roles.

>**Note:**

1. The **user** relationship in **UserRole** is a one to one relationship between **User** and **UserRole**.

2. The **role** relationship in **UserRole** is a one to one relationship between **Role** and **UserRole**.

3. The **roles** relationship in **User** is one to many relationship between **User** and its **roles**.

4. The **users** relationship in **Role** is one to many relationship between **Role** and its **users**.

*Dont worry if relationships are not very clear right now*

### Let's play with SqlAlchemy a little more.

**Getting a single row from database**

`>> user = User.query.get(1)`

This will give us the user with id 1. This query will return the first row that matches the condition.

`>> user = User.query.filter(User.id == 1).first()`

Same result as above, same query.

 `>> user = User.query.filter(User.id == 1).all()`

This will try to search the database for all the users with id one and will scan all the rows in the table. This will give us an array of users with id 1.

`>> user = User.query.filter(User.first_name='saurabh').first()`

Self explanatory

```python
>> from sqlalchemy import and_
>> user = User.query.join(UserRole, and_(UserRole.role_id==1, UserRole.user_id==User.id)).all()
```

  This will give user all the users which has role with id 1 associated with it. This query is running a join query between User and UserRole. To see what query sqlalchemy is running you can always do:
``` python
>> print(User.query.join(UserRole, and_(UserRole.role_id==1, UserRole.user_id==User.id)))
```

``` python
user = User.query.join(UserRole, and_(UserRole.user_id==User.id)).join(Role, and_(Role.id == UserRole.role_id, Role.name=='admin')).all()
```

Guess what this is doing?


**Have Fun!!**


[^rest api]: [RestApi]
