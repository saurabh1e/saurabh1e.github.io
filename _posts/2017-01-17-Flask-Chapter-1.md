Writing Restful Api's in Flask
===================


In this series of 10 Chapters we will learn basics of writing a **restful api**[^rest api] service using flask and create a seed app which can be used as building block for large web applications.  This series target audience with basic knowledge of using Flask or any other MVC framework.

----------


Chapter 1
-------------

### Starting Flask Server

    enter code here
    from flask import Flask
    app = Flask(__name__)

	@app.route('/')
	def hello_world():
	    return 'Hello, World!'

Save a file with the above code and start the server by running following commands.

    $ export FLASK_APP=<filename>
    $ flask run

###  Adding Database Model's


> **Note:**

> - Install SQLAlchemy and Flask-SQLAlchemy.
> - You must have sqlite installed and running.
	
We will be adding three models User, Role, UserRole. 

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


		def __init__(self, name, email):
	        self.name = name
	        self.email = email
	
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


Every model contains a **`id`** field which is used as a primary key and a **`created_on`** and **`updated_on`** which keeps track of when the row was created and when was it last time updated.
All the models also contains a **`__repr__()`** function which makes the objects more representable and alse help will debugging.

>**Note**:
> -  Printing an object of user model in console (`print(User())`) will print something like this `<id None name None>`
>  - After adding some data to the model 
>  
>>  `user = User()
> user.name = 'username'
> user.id = 1
> print(user)
> <id 1 user username>`



Lets see how we can make it more readable and concise by adding class BaseMixin and ReprMixin class.

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


By adding BaseMixin and ReprMixin we can put all the repeated code under one class and inherit those classes where ever required.
Our BaseMixin class contains three attributes **`id, created_on, updated_on`** which were common among all the models. It also contains a declarative attribute which automatically generates the table name using the class name, making code more readable, concise and less error prone.

Class ReprMixin helps us by adding a generic function which uses **`id`** and **`name`** by default attributes of class to represent the objects. The **`__repr__fields `** attribute can easily be overridden in sub classes as required.

Our new code looks like this

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
    
        def __init__(self,name, email):
            self.name = name
            self.email = email
    
    
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




Start a python  console and create your first object.

    $ python
    >> from app import *
    >> user = User('saurabh', 'saurabh.1e1@gmail.com')
    >> db.session.add(user)
    >> db.session.commit()
    >> user
    >> <User id=1 name=saurabh>
    


### Adding More Fields

Lets complete the our models, our goal is to create a complete user model to hold all the necessary data of user, a role model to have different roles in our web app and a model user role which acts as a junction between roles and user storing info about which user is connected to which role.

#### **Updated User Model:**

    class User(db.Model, BaseMixin, ReprMixin):
        __tablename__ = 'user'
    
        email = db.Column(db.String(120), unique=True)
        password = db.Column(db.String(255), unique=True)
        first_name = db.Column(db.String(40), nullable=False)
		last_name = db.Column(db.String(40))    
        profile_picture = db.Column(db.Text()))
       	active = db.Column(db.Boolean())
	    last_login_at = db.Column(db.DateTime())
		date_of_birth = db.Column(db.Date)

        def __init__(self,name, email):
            self.name = name
            self.email = email

		@hybrid_property
		def name(self):
			return '{}'.format(self.first_name) + (' {}'.format(self.last_name) if self.last_name else '')

>**Note:**
> - We have added two new columns **`first_name, last_name`** instead of name and introduced a hybrid property.
>> Now we can user name as a attribute, like **`user.name`**.




### Adding Relations

    class User(db.Model, BaseMixin, ReprMixin):
    	roles = db.relationship('Role', secondary='user_role', back_populates='users')

    class Role(db.Model, BaseMixin, ReprMixin):
    
    	users = db.relationship('User', secondary='user_role', back_populates='roles')
    	
    class UserRole(db.Model, BaseMixin, ReprMixin):
    
        user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
        role_id = db.Column(db.Integer, db.ForeignKey('role.id'))

		role = db.relationship('Role', foreign_keys=[role_id])
		user = db.relationship('User', foreign_keys=[user_id])


[^rest api]: [RestApi]sss
