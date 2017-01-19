<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hello!</title>
<link rel="stylesheet" href="https://stackedit.io/res-min/themes/base.css" />
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>
</head>
<body><div class="container"><h1 id="writing-restful-apis-in-flask">Writing Restful Api’s in Flask</h1>

<p>In this series of 10 Chapters we will learn basics of writing a <strong>restful api</strong><a href="#fn:rest-api" id="fnref:rest-api" title="See footnote" class="footnote">1</a> service using flask and create a seed app which can be used as building block for large web applications.  This series target audience with basic knowledge of using Flask or any other MVC framework.</p>

<hr>



<h2 id="chapter-1">Chapter 1</h2>



<h3 id="starting-flask-server">Starting Flask Server</h3>



<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-keyword">from</span> flask <span class="hljs-keyword">import</span> Flask
app = Flask(__name__)

<span class="hljs-decorator">@app.route('/')</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">hello_world</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">return</span> <span class="hljs-string">'Hello, World!'</span></code></pre>

<p>Save a file with the above code and start the server by running following commands.</p>



<pre class="prettyprint"><code class="language-bash hljs ">$ <span class="hljs-keyword">export</span> FLASK_APP=&lt;filename&gt;
$ flask run</code></pre>



<h3 id="adding-database-models">Adding Database Model’s</h3>

<blockquote>
  <p><strong>Note:</strong></p>
  
  <ul>
  <li>Install SQLAlchemy and Flask-SQLAlchemy.</li>
  <li>You must have sqlite installed and running.</li>
  </ul>
</blockquote>

<p>We will be adding three models User, Role, UserRole. </p>



<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-keyword">import</span> os
<span class="hljs-keyword">from</span> flask <span class="hljs-keyword">import</span> Flask
<span class="hljs-keyword">from</span> flask_sqlalchemy <span class="hljs-keyword">import</span> SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config[<span class="hljs-string">'SQLALCHEMY_DATABASE_URI'</span>] = <span class="hljs-string">'sqlite:///{}'</span>.format(os.path.join(basedir, <span class="hljs-string">'test.db'</span>))
db = SQLAlchemy(app)


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span><span class="hljs-params">(db.Model)</span>:</span>
    __tablename__ = <span class="hljs-string">'user'</span>

    id = db.Column(db.Integer, primary_key=<span class="hljs-keyword">True</span>)
    name = db.Column(db.String(<span class="hljs-number">80</span>))
    email = db.Column(db.String(<span class="hljs-number">120</span>), unique=<span class="hljs-keyword">True</span>, nullable=<span class="hljs-keyword">False</span>)
    created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())

    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__repr__</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> <span class="hljs-string">'&lt;id {} user {}&gt;'</span>.format(self.id, self.name)


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Role</span><span class="hljs-params">(db.Model)</span>:</span>
    __tablename__ = <span class="hljs-string">'role'</span>

    id = db.Column(db.Integer, primary_key=<span class="hljs-keyword">True</span>)
    name = db.Column(db.String(<span class="hljs-number">80</span>), unique=<span class="hljs-keyword">True</span>)
    description = db.Column(db.Text, unique=<span class="hljs-keyword">True</span>)
    created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())

    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__repr__</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> <span class="hljs-string">'&lt;id {} role {}&gt;'</span>.format(self.id, self.name)


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRole</span><span class="hljs-params">(db.Model)</span>:</span>
    __tablename__ = <span class="hljs-string">'user_role'</span>

    id = db.Column(db.Integer, primary_key=<span class="hljs-keyword">True</span>)
    user_id = db.Column(db.Integer, db.ForeignKey(<span class="hljs-string">'user.id'</span>))
    role_id = db.Column(db.Integer, db.ForeignKey(<span class="hljs-string">'role.id'</span>))
    created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())

    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__repr__</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> <span class="hljs-string">'&lt;user_id {} role_id {}&gt;'</span>.format(self.user_id, self.role_id)


<span class="hljs-decorator">@app.route('/')</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">hello_world</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">return</span> <span class="hljs-string">'Hello, World!'</span></code></pre>

<p>The user model and role models are self explanatory. <em>UserRole</em> model might look a little confusing to some, <br>
UserRole model is called an association table and it acts like a junction between <strong>User</strong> and <strong>Role</strong>. It stores the data of which user is connected to which role.</p>

<blockquote>
  <p><strong>Example:</strong></p>
  
  <table>
<thead>
<tr>
  <th align="center">id</th>
  <th align="center">user_id</th>
  <th align="center">role_id</th>
</tr>
</thead>
<tbody><tr>
  <td align="center">1</td>
  <td align="center">1</td>
  <td align="center">1</td>
</tr>
<tr>
  <td align="center">2</td>
  <td align="center">1</td>
  <td align="center">2</td>
</tr>
<tr>
  <td align="center">3</td>
  <td align="center">2</td>
  <td align="center">1</td>
</tr>
</tbody></table>

  
  <p>This shows that user with id 1 has two roles with id 1 and 2 associated with it and user with id 2 has one role with id 1 associated with it.</p>
</blockquote>

<p>Every model contains a <strong><code>id</code></strong> field which is used as a primary key and a <strong><code>created_on</code></strong> and <strong><code>updated_on</code></strong> which keeps track of when the row was created and when was it last time updated. <br>
All the models also contains a <strong><code>__repr__()</code></strong> function which makes the objects more representable and alse help will debugging.</p>

<blockquote>
  <p><strong>Note</strong>: <br>
  -  Printing an object of user model in console (<code>print(User())</code>) will print something like this <code>&lt;id None name None&gt;</code> <br>
   - Lets add some data <br>
   <code>&gt;&gt; user = User()</code> <br>
   <code>&gt;&gt; user.name = 'username'</code> <br>
   <code>&gt;&gt; user.id = 1</code> <br>
  - After adding some data it will look like this. <br>
   <code>&gt;&gt; print(user)</code> <br>
   <code>&gt;&gt; &lt;id 1 user username&gt;</code></p>
</blockquote>

<p>Lets see how we can make it more readable and concise by writting two more classes BaseMixin and ReprMixin.</p>



<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-keyword">import</span> re
<span class="hljs-keyword">from</span> sqlalchemy.ext.declarative <span class="hljs-keyword">import</span> declared_attr
<span class="hljs-keyword">from</span> sqlalchemy.schema <span class="hljs-keyword">import</span> Index

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">to_underscore</span><span class="hljs-params">(name)</span>:</span>

    s1 = re.sub(<span class="hljs-string">'(.)([A-Z][a-z]+)'</span>, <span class="hljs-string">r'\1_\2'</span>, name)
    <span class="hljs-keyword">return</span> re.sub(<span class="hljs-string">'([a-z0-9])([A-Z])'</span>, <span class="hljs-string">r'\1_\2'</span>, s1).lower()


 <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BaseMixin</span><span class="hljs-params">(object)</span>:</span>

     <span class="hljs-decorator">@declared_attr</span>
     <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__tablename__</span><span class="hljs-params">(self)</span>:</span>
         <span class="hljs-keyword">return</span> to_underscore(self.__name__)

     id = db.Column(db.Integer, primary_key=<span class="hljs-keyword">True</span>)
     created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
     updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReprMixin</span><span class="hljs-params">(object)</span>:</span>

    __repr_fields__ = [<span class="hljs-string">'id'</span>, <span class="hljs-string">'name'</span>]

    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__repr__</span><span class="hljs-params">(self)</span>:</span>
        fields = {f: getattr(self, f, <span class="hljs-string">'&lt;BLANK&gt;'</span>) <span class="hljs-keyword">for</span> f <span class="hljs-keyword">in</span> self.__repr_fields__}
        pattern = [<span class="hljs-string">'{0}={{{0}}}'</span>.format(f) <span class="hljs-keyword">for</span> f <span class="hljs-keyword">in</span> self.__repr_fields__]
        pattern = <span class="hljs-string">' '</span>.join(pattern)
        pattern = pattern.format(**fields)
        <span class="hljs-keyword">return</span> <span class="hljs-string">'&lt;{} {}&gt;'</span>.format(self.__class__.__name__, pattern)</code></pre>

<p>By adding BaseMixin and ReprMixin we can put all the repeated code under one class and inherit those classes where ever required. <br>
Our BaseMixin class contains three attributes <strong><code>id, created_on, updated_on</code></strong> which were common among all the models. It also contains a declarative attribute which automatically generates the table name using the class name, making code more readable, concise and less error prone.</p>

<p>Class ReprMixin helps us by adding a generic function which uses <strong><code>id</code></strong> and <strong><code>name</code></strong> by default attributes of class to represent the objects. The <strong><code>__repr__fields</code></strong> attribute can easily be overridden in sub classes as required.</p>

<p>Our new code looks like this</p>



<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-keyword">import</span> os
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">from</span> flask <span class="hljs-keyword">import</span> Flask
<span class="hljs-keyword">from</span> flask_sqlalchemy <span class="hljs-keyword">import</span> SQLAlchemy
<span class="hljs-keyword">from</span> sqlalchemy.ext.declarative <span class="hljs-keyword">import</span> declared_attr

basedir = os.path.abspath(os.path.dirname(__file__))
app = Flask(__name__)
app.config[<span class="hljs-string">'SQLALCHEMY_DATABASE_URI'</span>] = <span class="hljs-string">'sqlite:///{}'</span>.format(os.path.join(basedir, <span class="hljs-string">'test.db'</span>))
db = SQLAlchemy(app)
db.create_all()


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">to_underscore</span><span class="hljs-params">(name)</span>:</span>

    s1 = re.sub(<span class="hljs-string">'(.)([A-Z][a-z]+)'</span>, <span class="hljs-string">r'\1_\2'</span>, name)
    <span class="hljs-keyword">return</span> re.sub(<span class="hljs-string">'([a-z0-9])([A-Z])'</span>, <span class="hljs-string">r'\1_\2'</span>, s1).lower()


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BaseMixin</span><span class="hljs-params">(object)</span>:</span>

    <span class="hljs-decorator">@declared_attr</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__tablename__</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> to_underscore(self.__name__)

    id = db.Column(db.Integer, primary_key=<span class="hljs-keyword">True</span>)
    created_on = db.Column(db.TIMESTAMP, default=db.func.current_timestamp())
    updated_on = db.Column(db.TIMESTAMP, onupdate=db.func.current_timestamp())


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReprMixin</span><span class="hljs-params">(object)</span>:</span>

    __repr_fields__ = [<span class="hljs-string">'id'</span>, <span class="hljs-string">'name'</span>]

    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__repr__</span><span class="hljs-params">(self)</span>:</span>
        fields = {f: getattr(self, f, <span class="hljs-string">'&lt;BLANK&gt;'</span>) <span class="hljs-keyword">for</span> f <span class="hljs-keyword">in</span> self.__repr_fields__}
        pattern = [<span class="hljs-string">'{0}={{{0}}}'</span>.format(f) <span class="hljs-keyword">for</span> f <span class="hljs-keyword">in</span> self.__repr_fields__]
        pattern = <span class="hljs-string">' '</span>.join(pattern)
        pattern = pattern.format(**fields)
        <span class="hljs-keyword">return</span> <span class="hljs-string">'&lt;{} {}&gt;'</span>.format(self.__class__.__name__, pattern)


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
    __tablename__ = <span class="hljs-string">'user'</span>

    name = db.Column(db.String(<span class="hljs-number">80</span>))
    email = db.Column(db.String(<span class="hljs-number">120</span>), unique=<span class="hljs-keyword">True</span>)

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Role</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
    __tablename__ = <span class="hljs-string">'role'</span>

    name = db.Column(db.String(<span class="hljs-number">80</span>), unique=<span class="hljs-keyword">True</span>)
    description = db.Column(db.Text, unique=<span class="hljs-keyword">True</span>)


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRole</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
    __tablename__ = <span class="hljs-string">'user_role'</span>
    __repr_fields__ = [<span class="hljs-string">'user_id'</span>, <span class="hljs-string">'role_id'</span>]

    user_id = db.Column(db.Integer, db.ForeignKey(<span class="hljs-string">'user.id'</span>))
    role_id = db.Column(db.Integer, db.ForeignKey(<span class="hljs-string">'role.id'</span>))


<span class="hljs-decorator">@app.route('/')</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">hello_world</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">return</span> <span class="hljs-string">'Hello, World!'</span>
</code></pre>

<p>Start a python  console and create your first object.</p>



<pre class="prettyprint"><code class="language-bash hljs ">$ python</code></pre>



<pre class="prettyprint"><code class="language-python hljs ">&gt;&gt; <span class="hljs-keyword">from</span> app <span class="hljs-keyword">import</span> *
&gt;&gt; user = User(<span class="hljs-string">'saurabh'</span>, <span class="hljs-string">'saurabh.1e1@gmail.com'</span>)
&gt;&gt; db.session.add(user)
&gt;&gt; db.session.commit()
&gt;&gt; user
&gt;&gt; &lt;User id=<span class="hljs-number">1</span> name=saurabh&gt;
</code></pre>



<h3 id="adding-more-fields">Adding More Fields</h3>

<p>Lets complete the our models. Our goal is to create a database to hold all the necessary data of a user, all the available roles and info of different roles associated with the users.</p>



<h4 id="updated-user-model"><strong>Updated User Model:</strong></h4>



<pre class="prettyprint"><code class="language-python hljs ">
<span class="hljs-keyword">from</span> sqlalchemy.ext.hybrid <span class="hljs-keyword">import</span> hybrid_property

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
    __tablename__ = <span class="hljs-string">'user'</span>

    email = db.Column(db.String(<span class="hljs-number">120</span>), unique=<span class="hljs-keyword">True</span>)
    password = db.Column(db.String(<span class="hljs-number">255</span>))
    first_name = db.Column(db.String(<span class="hljs-number">40</span>), nullable=<span class="hljs-keyword">False</span>)
    last_name = db.Column(db.String(<span class="hljs-number">40</span>))    
    profile_picture = db.Column(db.Text()))
    bio = db.Column(db.Text()))
    active = db.Column(db.Boolean())
    last_login_at = db.Column(db.DateTime())
    date_of_birth = db.Column(db.Date)

    <span class="hljs-decorator">@hybrid_property</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">name</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> <span class="hljs-string">'{}'</span>.format(self.first_name) + (<span class="hljs-string">' {}'</span>.format(self.last_name) <span class="hljs-keyword">if</span> self.last_name <span class="hljs-keyword">else</span> <span class="hljs-string">''</span>)
</code></pre>

<blockquote>
  <p><strong>Note:</strong> <br>
  - We have added two new columns <strong><code>first_name, last_name</code></strong> instead of name and introduced a hybrid property. <br>
  - Now we can use name as a attribute of user class instance, like <strong><code>user.name</code></strong>.</p>
</blockquote>



<h3 id="adding-relations">Adding Relations</h3>

<p><strong>So what are relations?</strong> <br>
Database tables are often related to one another, SqlAlchemy relations makes managing and working with these relationships easy. Defining relations powerful method chaining and querying capabilities.</p>

<p>To know more about what sqlalchemy relationships are read <a href="http://www.ergo.io/blog/sqlalchemy-relationships-from-beginner-to-advanced/">this</a> and <a href="http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html">this</a>.</p>

<p><strong>User Model</strong></p>



<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
    roles = db.relationship(<span class="hljs-string">'Role'</span>, secondary=<span class="hljs-string">'user_role'</span>, back_populates=<span class="hljs-string">'users'</span>)</code></pre>

<p><strong>Role Model</strong></p>



<pre class="prettyprint"><code class="language-python hljs ">  <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Role</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
     users = db.relationship(<span class="hljs-string">'User'</span>, secondary=<span class="hljs-string">'user_role'</span>, back_populates=<span class="hljs-string">'roles'</span>)</code></pre>

<p><strong>UserRole Model</strong></p>



<pre class="prettyprint"><code class="language-python hljs "><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRole</span><span class="hljs-params">(db.Model, BaseMixin, ReprMixin)</span>:</span>
    user_id = db.Column(db.Integer, db.ForeignKey(<span class="hljs-string">'user.id'</span>))
    role_id = db.Column(db.Integer, db.ForeignKey(<span class="hljs-string">'role.id'</span>))
    role = db.relationship(<span class="hljs-string">'Role'</span>, foreign_keys=[role_id])
    user = db.relationship(<span class="hljs-string">'User'</span>, foreign_keys=[user_id])</code></pre>

<p>We have added a relation <strong><code>roles</code></strong> in user model and <strong><code>users</code></strong> in role model, this now allows us to access, add and remove and update a user’s roles from a user object and vice a versa.</p>

<blockquote>
  <p><strong>Example:</strong></p>
</blockquote>

<p><strong>Adding an user and a role</strong>.</p>

<p><code>&gt;&gt; user = User()</code></p>

<p><code>&gt;&gt; user.first_name = 'saurabh'</code></p>

<p><code>&gt;&gt; user.email = 'example@gmail.com'</code></p>

<p><code>&gt;&gt; db.session..add(user)</code></p>

<p><code>&gt;&gt; db.session.commit()</code></p>

<p><code>&gt;&gt; role = Role()</code></p>

<p><code>&gt;&gt; role.name = 'admin'</code></p>

<p><code>&gt;&gt; db.session..add(role)</code></p>

<p><code>&gt;&gt; db.session.commit()</code></p>

<p><strong>Now we can do this</strong></p>

<p><code>&gt;&gt; user.roles.append(role)</code></p>

<p><code>&gt;&gt; db.session.commit()</code></p>

<p><code>&gt;&gt; print(user.roles)</code></p>

<p><code>&gt;&gt; [&lt;Role id=1 name=admin&gt;]</code></p>

<p><code>&gt;&gt; print(role.users)</code></p>

<p><code>&gt;&gt; [&lt;User id=1 name=saurabh&gt;]</code></p>

<p>Our UserRole model also contains two relationships which are acting as junction between roles relation in user and users relationship in roles.</p>

<blockquote>
  <p><strong>Note:</strong></p>
</blockquote>

<ol>
<li><p>The <strong>user</strong> relationship in <strong>UserRole</strong> is a one to one relationship between <strong>User</strong> and <strong>UserRole</strong>.</p></li>
<li><p>The <strong>role</strong> relationship in <strong>UserRole</strong> is a one to one relationship between <strong>Role</strong> and <strong>UserRole</strong>.</p></li>
<li><p>The <strong>roles</strong> relationship in <strong>User</strong> is one to many relationship between <strong>User</strong> and its <strong>roles</strong>.</p></li>
<li><p>The <strong>users</strong> relationship in <strong>Role</strong> is one to many relationship between <strong>Role</strong> and its <strong>users</strong>.</p></li>
</ol>

<p><em>Dont worry if relationships are not very clear right now</em></p>



<h3 id="lets-play-with-sqlalchemy-a-little-more">Let’s play with SqlAlchemy a little more.</h3>

<p><strong>Getting a single row from database</strong></p>

<p><code>&gt;&gt; user = User.query.get(1)</code></p>

<p>This will give us the user with id 1. This query will return the first row that matches the condition.</p>

<p><code>&gt;&gt; user = User.query.filter(User.id == 1).first()</code></p>

<p>Same result as above, same query.</p>

<p><code>&gt;&gt; user = User.query.filter(User.id == 1).all()</code></p>

<p>This will try to search the database for all the users with id one and will scan all the rows in the table. This will give us an array of users with id 1.</p>

<p><code>&gt;&gt; user = User.query.filter(User.first_name='saurabh').first()</code></p>

<p>Self explanatory</p>



<pre class="prettyprint"><code class="language-python hljs ">&gt;&gt; <span class="hljs-keyword">from</span> sqlalchemy <span class="hljs-keyword">import</span> and_ 
&gt;&gt; user = User.query.join(UserRole, and_(UserRole.role_id==<span class="hljs-number">1</span>, UserRole.user_id==User.id)).all()</code></pre>

<p>This will give user all the users which has role with id 1 associated with it. This query is running a join query between User and UserRole. To see what query sqlalchemy is running you can always do:</p>



<pre class="prettyprint"><code class="language-python hljs ">&gt;&gt; print(User.query.join(UserRole, and_(UserRole.role_id==<span class="hljs-number">1</span>, UserRole.user_id==User.id)))</code></pre>



<pre class="prettyprint"><code class="language-python hljs ">user = User.query.join(UserRole, and_(UserRole.user_id==User.id)).join(Role, and_(Role.id == UserRole.role_id, Role.name==<span class="hljs-string">'admin'</span>)).all()</code></pre>

<p>Guess what this is doing?</p>

<p><strong>Have Fun!!</strong></p>

<div class="footnotes"><hr><ol><li id="fn:rest-api">[RestApi] <a href="#fnref:rest-api" title="Return to article" class="reversefootnote">↩</a></li></ol></div></div></body>
</html>