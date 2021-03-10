# flask-sqlAlchemy

1. app.py (DML):

   - [query_database](#query_database)
   - [create_instance_and_commit](#create_instance_and_commit)
   - [update_instance](#update_instance)
   - [delete_instance](#delete_instance)
   - [rollback](#rollback)
   - [setup](#app.py)

2. models.py (DDL):
   - [explicit_inner_joins](##explicit_joins)
   - [explicit_outer_joins](##explicit_outer_joins)
     Relationships:
   - [one_to_one](##one_to_one_syntax)
   - [one_to_many](##one_to_many)
   - [many_to_many](##many_to_many)
   - [append_to_relationship](##append_to_relationship)
     setup:
   - [setup](##models_setup)
3. seed.py
   - [seed_file](#seed_file)
4. templates
   - [templates](#jinja_templates)
   - [render_wtform](##jinja_wtform)
   - [loop](##jinja_loops)
   - [if](##jinja_if)
5. forms
   - [WTF_forms](#WTForm)
   - [form-session-flow](###form-session-flow)
   - [form](##forms)
6. authentication
   - [authentication_authorization](##authentication_and_authorization)
   - [models.py](####models.py)
   - [app.py](####aapp.py)
   - [templates](####templates)
   - [forms.py](####forms.py)
7. API
   - [API_requests](#API)
8. flash
   - [flash](##flash)
9. Cookies and session
   - [cookies](##COOKIES)
   - [sessions-cookies](##flask_sessions)
10. tests.py
    - [tests](#tests)
    - [model_test](##model_test)
    - [views_test](##views_test)
11. debug
    - [debug_toolbar_extension](#debug)
12. [routes_demo](#routes_demo)
13. [install](#install)
    [setup](#setup)

# flask

9. [routes](#basics)
10. [redirect](##redirect)
11. [jsonify](##jsonify)
12. [folders_structure](##dir)
13. [flask_setup](#setup)

# query_database

- check if a `__repr__` method is defined in table declaration in models.py.

1. Fetching records:
   - `.get(pk)`
   - `.get_or_404(pk)`
   - `.all()` all records that match
   - `.first()` first record that match or **None** if nothing matches
   - `.one()` get first record, **Error** if no results or if there's more than one result, "there should be only one thing that matches this, otherwise give me error".
   - `.one_or_none()` **Error** if more than one result, **None** if 0 results.
   - `.count()` number of records found without fetching them.
2. Single queries:

   ```python
    # select specific columns (retrieves tuples):
    db.session.query(Employee.id, Employee.name).all() # [(1, 'Leonard'), (2, 'Liz'), (3, 'Maggie')]

   Pet.query.all() #(it's like 'SELECT * FROM pets')

   # GET
   Pet.query.get(1) # select by id (SELECT * FROM pets WHERE id = 1)
   Department.query.get('finance')

   # GET_OR_404
   Pet.query.get_or_404(7)
   Department.query.get_or_404('marketing')

   # FILTER_BY
   Pet.query.filter_by(species='dog').all() # all objects that matches 'dog'
   Pet.query.filter_by(species='dog').first() # only first result matching 'dog'
   Pet.query.filter_by(species='dog', hunger=12).all() # multiple paramsdcxc

   # FILTER
   # for queries where we search this OR that, this AND that, this uppercase OR lowercase
   Pet.query.filter(Pet.species == 'dog').all()
   Pet.query.filter(Pet.hunger < 20).all()
   Pet.query.filter(Pet.hunger > 20).all()
   Pet.query.filter(Pet.hunger != 20).all()
      #AND
   Pet.query.filter(Pet.hunger < 15, Pet.species == 'dog').all()
   Pet.query.filter((Pet.hunger < 15) & (Pet.species == 'dog')).all()
     # OR_
   Employee.query.filter(db.or_(Employee.state == 'CA', Employee.id > 65)).all()
   Employee.query.filter((Employee.state == 'CA') | (Employee.id > 65) ).all()

   #LIKE / ILIKE
   Employee.query.filter(Employee.name.like('%Jane%')).first()
   Employee.query.filter(Employee.name.ilike('%jane%')).all()
   # IN_
   Employee.query.filter(Employee.id.in_([22, 23, 24])).one()

   # GROUP BY / HAVING
    q = Employee.query

    q.group_by('state')
    q.group_by('state').having(db.func.count(Employee.id) > 2)

   #ORDER BY
    q.order_by('state')

   #OFFSET
    q.offset(10)

   #LIMIT
   q.limit(10)
   ```

   [methods_full_list_docs](https://docs.sqlalchemy.org/en/13/orm/query.html#sqlalchemy.orm.query.Query.offset)

3. Chained queries:

   ```python
   new_hires = Employee.query.filter(Employee.id >= 4)
   cal_new_hires = new_hires.filter(Employee.state == 'CA')
   new_hires.all() # [<Employee 3>, <Employee 12>, <Employee233>]
   cal_new_hires.all() # [<Employee 3>, <Employee 12>]
   ```

# create_instance_and_commit

1. (Create table in models.py.)
2. in app.py:

   ```python
   from models import db, connect_db, Pet

   # CREATE NEW INSTANCES (ONE BY ONE)
   fluffy = Pet(name='Fluffy', species='Pet', hunger=13)
   rocket = Pet(name='Rocket', species='Cat', hunger=10)
   toby = Pet(name='Toby', species='Goose', hunger=20)

   # CREATE NEW INSTANCES (MULTIPLE AT ONCE)
   names = ['Sushi', 'Scout', 'Piggie', 'Carrot']
   species = ['pig', 'cat', 'bunny', 'bunny']
   pets = [Pet(name=n, species=s) for n, s in zip(names, species)]
   pets[0].name  # Sushi

   # STAGE NEW INSTANCES (ONE BY ONE):
   db.session.add(fluffy)

   # STAGE NEW INSTANCES (ALL AT ONCE):
   db.session.add_all(pets)
   #or
   db.session.add_all([d1, d2, d3])

   # COMMIT NEW INSTANCES:
   db.session.commit()
   ```

# update_instance

```python
rocket.name = 'Rocko'
db.session.add(rocket)
db.session.commit()
```

# delete_instance

```python
# DELETE SINGLE:
Pet.query.filter_by(hunger=23).delete()
db.session.commit()

# DELETE MULTIPLE:
Pet.query.filter(Pet.species == 'bunny').delete()
db.session.commit()
```

# rollback

```python
db.session.rollback()
```

When there's an error in the attempt to commit a new object to the database, we have to rollback before trying to add or update new data to db, this will get rid of anything that might have happened to the database session:

# models

## explicit_joins

Default inner join (returns only the data where both tables overlap, leaves outside columns that are in one table only)
To connect tables that hs foreign key constrains defined but not relationships. Or for outer joins.

> returns a list of tuples

```python
ex1 = db.session.query(Employee.name, Department.phone).join(Department).all()
ex1 # [('River Bottom', '232321321'), ('Lucas Luke', '232321321'), ('Sumer winter', '232321321')]

ex3 = db.session.query(Department.dept_code,
                       Employee.name).join(Employee).all()
ex3 # [('MKTG', 'River Bottom'), ('FNNC', 'Lucas Luke'), ('MKTG', 'Sumer winter')]


example = (db.session.query(Employee.name, Department.dept_name,
                            Department.phone).join(Department).all())
example # [('River Bottom', 'Marketing', '232321321'), ('Lucas Luke', 'Finance', '232321321'), ('Sumer winter', 'Marketing', '232321321')]

# JOIN + METHOD
ex4 = db.session.query(Department.dept_code,
                       Employee.name).join(Employee).filter(Employee.state == 'AL').all()
ex4 # [('FNNC', 'Lucas Luke')]

# RETRIEVE FULL MODELS:
ex5 = db.session.query(Department, Employee).join(Employee).all()
ex5 #[(<Department MKTG>, <Employee 1>), (<Department FNNC>, <Employee 2>), (<Department MKTG>, <Employee 3>)]

```

## explicit_outer_joins

Return all data, overlaping in tables or not.

```python
out1 = db.session.query(Employee.name, Department.dept_name,
                        Department.phone).outerjoin(Department).all()
out1  # [('River Bottom', 'Marketing', '232321321'), ('Arthur Miller', None, None), ('Mike Spencer', None, None)]

out_full_method = db.session.query(
    Employee, Department).outerjoin(Department).all()
out_full_method # [(<Employee 1>, <Department MKTG>), (<Employee 6>, None), (<Employee 7>, None)]
```

## one_to_one_syntax

```python
from flask_sqlalchemy import SQLAlchemy
# 1. Configure the connection to the database
db = SQLAlchemy()
# 2. A function to Associate the app with the database:
def connect_db(app):
    db.app = app
    db.init_app(app)
# 3. Define model (o sea table) (class name in Singular, table name in plural):

class Pet(db.Model):
    """Pet."""
    __tablename__ = "pets"

    # TABLE COLUMNS AND CONSTRAINS:
    id = db.Column(db.Integer,  # db.Integer same as INT
                   primary_key=True,  # PRIMARY KEY
                   autoincrement=True)
    name = db.Column(db.String(50),  # db.string same as TEXT (max length 50)
                     nullable=False,  # nullable false same as NOT NULL
                     unique=True)  # unique true same as UNIQUE
    species = db.Column(db.String(30), nullable=True)
    hunger = db.Column(db.Integer, nullable=False, default=20)  # default value
    available = db.Column(db.Boolean, nullable=False, default=True)

    # REPRESENTATION OBJECT:
    def __repr__(self):
        """show info about pet."""
        p = self
        return f"<Pet id={p.id} name={p.name} species={p.species} hunger={p.hunger}>"

    # CUSTOM METHODS:
    @classmethod
    def get_by_species(cls, species):
        return cls.query.filter_by(species=species).all()

    @classmethod
    def get_all_hungry(cls):
        return cls.query.filter(Pet.hunger > 15).all()

    def greet(self):
        """Greet using name"""
        return f"I'm {self.name} the {self.species or 'thing'}"

    def feed(self, food=10):
        """Updates hunger based on passed food. Default food is 10"""
        self.hunger -= food
        self.hunger = max(self.hunger, 0)


# CALL CLASS METHOD:
# in app.py:
Pet.get_by_species('cat')  # [Pet id=4 name=Scout]
Pet.get_all_hungry()

# CALL INSTANCE METHODS:
# In app.py:
piggie = Pet.query.get(6)
print(piggie.greet())  # I'm Piggie the bunny

# CALL METHOD AND UPDATE DATABASE:
# in app.py
fluffy = Pet.query.get(1)
fluffy.feed(9)
db.session.add(fluffy)
db.session.commit()
# now check database: fluffy.hunger: 4
```

## one_to_many

SQLAlchemy allows to create RELATIONSHIPS, to join different tables. First define a Foreign key constrain and then define the realtionship:
![db_graphic](/images/one-to-many-ex.jpg)

```python
from enum import unique
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import backref

db = SQLAlchemy()


def connect_db(app):
    db.app = app
    db.init_app(app)


class Department(db.Model):
    """Department. A department has many employees."""

    __tablename__ = "departments"

    dept_code = db.Column(db.Text, primary_key=True)
    dept_name = db.Column(db.Text,
                          nullable=False,
                          unique=True)
    phone = db.Column(db.Text)

    # RELATIONSHIP TO FOREIGN TABLE (if no backref somewhere else):
    # dept_employees = db.relationship('Employee')


class Employee(db.Model):
    """ employee table """

    __tablename__ = 'employees'

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.Text, nullable=False, unique=True)
    state = db.Column(db.Text, nullable=False, default='CA')

    # 1. FOREIGN KEY constraint:
    dept_code = db.Column(db.Text, db.ForeignKey('departments.dept_code'))

    # 2. RELATIONSHIP TO FOREIGN TABLE:
    # syntax "one way only"
    # department = db.relationship('Department')
    # syntax to go both ways (from Employee to Department and from Department to Employee, and
    # no need to add a relationship in Department):
    department_where_works = db.relationship('Department', backref='emps')


# USE RELATIONSHIP FROM DEPARTMENT:
# app.py
d = Department.query.get('MKTG')
d.dept_employees[0].name  # River Bottom
d.dept_employees  # [<Employee 1>, <Employee 3>, <Employee 5>]

# USE RELATIONSHIP FROM EMPLOYEE:
# app.py:
emp = Employee.query.get(1)
emp.department.dept_code  # MKTG
emp.department.phone # 23232323

# USE BACKREF RELATIONSHIP from Department:
d = Department.query.get('MKTG')
d.emps # [<Employee 1>, <Employee 3>, <Employee 5>]

# USE BACKREF RELATIONSHIP from Employee:
ca = Employee.query.filter_by(state='CA').first()
ca.name  # Summer Winter
ca.department_where_works  # Department MKTG
ca.department_where_works.phone  # 23232323
```

## many_to_many

![graphic](/images/m-m-example.jpg)

```python
class Project(db.Model):
    __tablename__ = 'projects'

    proj_code = db.Column(db.Text, primary_key=True)
    proj_name = db.Column(db.Text, nullable=False, unique=True)

    # RELATIONSHIP:
    proj_assignments = db.relationship(
        'EmployeeProject', backref='project_info')

# MIDDLE TABLE WITH CASCADE DELETION:
class PostTag(db.Model):
    """relates posts with tags"""
    __tablename__ = 'posts_tags'
    post_id = db.Column(db.Integer, db.ForeignKey(
        'posts.id', ondelete='cascade'), primary_key=True)
    tag_id = db.Column(db.Integer, db.ForeignKey('tags.id'), primary_key=True)

class EmployeeProject(db.Model):
    __tablename__ = 'employees_projects'
# Double primary key constraint:
    emp_id = db.Column(db.Integer, db.ForeignKey(
        'employees.id'), primary_key=True)
    proj_code = db.Column(db.Text, db.ForeignKey(
        'projects.proj_code'), primary_key=True)
    role = db.Column(db.Text)

class Employee(db.Model):
    """ employee table """
    __tablename__ = 'employees'

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.Text, nullable=False, unique=True)
    state = db.Column(db.Text, nullable=False, default='CA')

    # 1. FOREIGN KEY constraint:
    dept_code = db.Column(db.Text, db.ForeignKey(
        'departments.dept_code'), nullable=True)

    # 2. RELATIONSHIP:
    # syntax "one way only"
    # department = db.relationship('Department')
    # syntax to go both ways (from Employee to Department and from Department to Employee, and
    # no need to add a relationship in Department)
    # DIRECT RELATIONSHIPS (ONE TABLE TO ANOTHER):
    department_where_works = db.relationship('Department', backref='emps')
    assignments = db.relationship('EmployeeProject', backref='employee_info')
    # THROUGH RELATIONSHIP:
    # COMBINES THE THREE TABLES, THE 'SECONDARY' TABLE CONTAINS THE OTHER TWO TABLES.
    projects = db.relationship(
        'Project', secondary='employees_projects', backref='employees')

#USE RELATIONSHIPS
river = EmployeeProject.query.filter_by(emp_id=1).first()
details = river.employee_info
print(details.department_where_works)

# USE THROUGH RELATIONSHIP:
river = Employee.query.get(1)
river_projs = river.projects
river_projs # [<Project sps>, <Project csc>]

car = Project.query.get('csc')
car.employees  # [<Employee 1>, <Employee 5>, <Employee 6>]
```

## append_to_relationship

THROUGH RELATIONSHIP MUST BE DEFINED ALREADY.

```python
#Example 1:
# grab employee from db:
nadine = Employee.query.get(8)
# grab project from db:
bsb_project = Project.query.get('bsb')
# append the emp to the project:
nadine.projects.append(bsb_project)
# commit the operations:
db.session.commit()
nadine.projects  # [<Project bsb>]

# Example 2:
# grab project from db:
bsb_proj = Project.query.get('bsb')
# check project's employees using the through relationship:
print(bsb_proj.employees)  # [<Employee 3>]
# grab employee to be added:
arthur = Employee.query.get(6)
# append employee to project:
bsb_proj.employees.append(arthur)
# commit changes:
db.session.commit()
# check employees for project:
print(bsb_proj.employees)  # [<Employee 3>, <Employee 9>, <Employee 10>]

# Example 3, create and append:
loyola = Employee(name='Marta Loyola', state='BA', dept_code='FNNC')
loyola.assignments.append(EmployeeProject(proj_code='csc', role='director'))
db.session.add(loyola)
db.session.commit()
```

## models_setup

models.py:

```python

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def connect_db(app):
    db.app = app
    db.init_app(app)

class Pet(db.Model):
    """pet available for adoption"""
    __tablename__ = "pets"

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.String(20), nullable=False)
    species = db.Column(db.String(20), nullable=False)
    photo_url = db.Column(db.String)
    age = db.Column(db.Integer)
    notes = db.Column(db.String)
    available = db.Column(db.Boolean, nullable=False, default=True)

```

3. Write models.
4. check venv active
5. check postgres server active
6. In app.py:
   ```python
   db.create_all()
   ```
7. in terminal: `python app.py`
8. Once the db is created, erase 'db.create_all()`, otherwise will create a new table every time 'app.py' runs.
9. In app.py, import the new created class:
   ```python
   from models import db, connect_db, Mymodel
   ```

---

# update_model

Go to terminal, connect to database and drop table with SQL syntax (`DROP TABLE my_table`). Then redo the table in Alchemy and run it again from app.pyYo .
To reset all tables:

```python
db.drop_all()
db.create_all()
```

# seed_file

```python
"""Seed file to make sample data for pets db."""

from models import Pet, db
from app import app

# Create all tables
db.drop_all()
db.create_all()

# If table isn't empty, empty it
Pet.query.delete()

# Add pets
whiskey = Pet(name='Whiskey', species="dog")
bowser = Pet(name='Bowser', species="dog", hunger=10)
spike = Pet(name='Spike', species="porcupine")

# Add new objects to session, so they'll persist
db.session.add(whiskey)
db.session.add(bowser)
db.session.add(spike)

# Commit to database
db.session.commit()

# run file: 'python seed.py', beware database will be erased
```

# app.py

An app.py file demo with sqlalchemy queries:

```python
from flask import Flask, request, render_template, redirect
from flask.helpers import flash
# 1. Import models.py:
from models import db, connect_db, Pet

app = Flask(__name__)
# 2. Database config:
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql:///my_database'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SQLALCHEMY_ECHO'] = True
# Other configs:
app.config['SECRET_KEY'] = 'caca'

# 3. Call db:
connect_db(app)

# routes:
@app.route('/')
def list_pets():
    """shows list of pets in database"""
    pets = Pet.query.all()
    print(pets)
    return render_template('list.html', pets=pets)


@app.route('/', methods=["POST"])
def create_pet():
    # capture data from form:
    name = request.form["name"]
    species = request.form["species"]
    hunger = request.form["hunger"]
    hunger = int(hunger) if hunger else None

    # Create model class:
    new_pet = Pet(name=name, species=species, hunger=hunger)

    # add to db session:
    db.session.add(new_pet)

    # try to commit or catch error and rollback:
    try:
        db.session.commit()
    except Exception as error:
        db.session.rollback()
        return 'No good, do something!!!!'

    return redirect('/')


@app.route('/delete/<int:pet_id>')
def delete_pet(pet_id):
    Pet.query.filter_by(id=pet_id).delete()
    try:
        db.session.commit()
    except Exception as error:
        db.session.rollback()
        return "No good coudln't delete"
    return redirect('/')


@app.route('/species/<species_name>')
def show_pets_by_species(species_name):
    pets = Pet.get_by_species(species_name)
    return render_template('species.html', pets=pets, species_name=species_name)


@app.route('/<int:pet_id>')
def show_pet(pet_id):
    """show details about single pet"""
    pet = Pet.query.get_or_404(pet_id)
    return render_template('details.html', pet=pet)
```

---

# tests

USE A SEPARATE DATABASE FOR TESTS!!!!!! Create a mock one, don't test on app database.

## model_test

demo:

```python
# 1. Create alternate database for tests
# 2:

from unittest import TestCase

from app import app
from models import db, Pet

# Use test database and don't clutter tests with SQL
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql:///test_pets_db'
app.config['SQLALCHEMY_ECHO'] = False

db.drop_all()
db.create_all()


class PetModelTestCase(TestCase):
    """Tests for model for Pets."""

    def setUp(self):
        """Clean up any existing pets."""

        Pet.query.delete()

    def tearDown(self):
        """Clean up any fouled transaction."""

        db.session.rollback()

    def test_greet(self):
        pet = Pet(name="TestPet", species="dog", hunger=10)
        self.assertEquals(pet.greet(), "I'm TestPet the dog")

    def test_feed(self):
        pet = Pet(name="TestPet", species="dog", hunger=10)
        pet.feed(5)
        self.assertEquals(pet.hunger, 5)

        pet.feed(999)
        self.assertEquals(pet.hunger, 0)

    def test_get_by_species(self):
        pet = Pet(name="TestPet", species="dog", hunger=10)
        db.session.add(pet)
        db.session.commit()

        dogs = Pet.get_by_species('dog')
        self.assertEquals(dogs, [pet])


# RUN TESTS:
# 'python -m unittest test_models.py'

```

## views_test

demo:

```python
from unittest import TestCase

from werkzeug.utils import html

from app import app
from models import db, Pet

# Use test database and don't clutter tests with SQL
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql:///test_pets_db'
app.config['SQLALCHEMY_ECHO'] = False

# Make Flask errors be real errors, rather than HTML pages with error info
app.config['TESTING'] = True

# This is a bit of hack, but don't use Flask DebugToolbar
app.config['DEBUG_TB_HOSTS'] = ['dont-show-debug-toolbar']

db.drop_all()
db.create_all()


class PetViewsTestCase(TestCase):
    """Tests for views for Pets."""

    def setUp(self):
        """Add sample pet."""

        Pet.query.delete()

        pet = Pet(name="TestPet", species="dog", hunger=10)
        db.session.add(pet)
        db.session.commit()

        self.pet_id = pet.id

    def tearDown(self):
        """Clean up any fouled transaction."""

        db.session.rollback()

    def test_list_pets(self):
        with app.test_client() as client:
            resp = client.get("/")
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertIn('TestPet', html)

    def test_show_pet(self):
        with app.test_client() as client:
            resp = client.get(f"/{self.pet_id}")
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertIn('<h1>TestPet</h1>', html)
            # self.assertIn(self.pet.species, html)

    def test_create_pet(self):
        with app.test_client() as client:
            d = {"name": "TestPet2", "species": "cat", "hunger": 20}
            resp = client.post("/", data=d, follow_redirects=True)
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertIn("TestPet2", html)

    def test_show_pets_by_species(self):
        with app.test_client() as client:
            more_names = ['TestPet3', 'TestPet4', 'TestPet5']
            more_species = ['cat', 'cat', 'cat']
            more_pets = [Pet(name=n, species=s)
                         for n, s in zip(more_names, more_species)]
            db.session.add_all(more_pets)
            db.session.commit()
            resp = client.get(f"/species/cat")
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertNotIn('dog', html)

    def test_delete_pet(self):
        with app.test_client() as client:
            more_names = ['TestPet6', 'TestPet7', 'TestPet8']
            more_species = ['cat', 'cat', 'cat']
            more_pets = [Pet(name=n, species=s)
                         for n, s in zip(more_names, more_species)]
            db.session.add_all(more_pets)
            db.session.commit()

            resp = client.get(
                f"/delete/{more_pets[0].id}", follow_redirects=True)
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertNotIn(f"{more_pets[0]}", html)

```

# general

[data_types_list](https://docs.sqlalchemy.org/en/13/core/type_basics.html)

The point of a model is creating a class that will represent a table in theSQL database, so we canmarian interact with it through class methods andinstance methods. Each class points to each table. When defining a model, wedefine the SQL table that SQAlchemy will create for us.

# install

1. cd to project's folder.
2. activate virtual environment.
3. (install flask)
4. `pip install psycopg2-binary`
5. `pip install flask-sqlalchemy`

# setup

0. (`createdb name_of_db`)
1. run PostgreSQL server
2. `psql`
3. connect to database (`\c my_db`)
4. models.py:

```python
 from flask_sqlalchemy import SQLAlchemy

#1. Configure the connection to the database
db = SQLAlchemy()

#2. A function to Associate the app with the database:
def connect_db(app):
    db.app = app
    db.init_app(app)
```

5. app.py:

```python
from flask import Flask, request, render_template, redirect
# 1. Import models.py:
from models import db, connect_db, My_model_name
app = Flask(__name__)
# 2. Database config:
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql:///movies_example'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SQLALCHEMY_ECHO'] = True
# Other configs:
app.config['SECRET_KEY'] = 'caca'
# 3. Call db:
connect_db(app)
# check if it's connecting to the database:
actors = db.session.execute('SELECT * FROM actors')
print(list(actors))
```

6. Once models are written, to commit them in database, run in
   app.py or in seed.py:

```python
db.create_all()
```

Mind running only once, otherwise it will "seed" a new table every time.

# ORM

(Object-relational-mapping)
The idea of this is to creal a virtual object oriented database that allows us to interact with the database without writing SQL, and writing python or JS instead.
SQLAlchemy is an ORM (object-relational mapping). Python-based. It's a library that we have to install. We have a connection to a database, and instead of writing SQL directly in flask or node or java, we map that SQL information and queries into objects. On these objects we can then call methods on.
We can use SQLAlchemy with django or with python only. To use it with flask, there's a special package: `flask-sqlalchemy`, along with a tool called `psycopg2-binary`, that adapts flask-sqlalchemy to PostgreSQL

Connects the flask application with the SQL database.

---

# basics

full syntax:

```python
from flask import Flask, request, render_template, redirect, flash, jsonify
from flask_debugtoolbar import DebugToolbarExtension
from random import randint, choice, sample

# main instance of Flask class:
app = Flask(__name__)
# debug toolbar extansion config:
app.config['SECRET_KEY'] = "mypassword"
debug = DebugToolbarExtension(app)
# to stop debugger:
app.config['DEBUG_TB_INTERCEPT_REDIRECTS'] = False


# Reload with no cache for styling purposes:
@app.after_request
def apply_caching(response):
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    return response


# Root route
@app.route('/')
def home_page():
    """these functions are called 'view functions', because
    they set the rendering of the response"""
    return render_template('hello.html')

# Manage forms:
# 1- Display form:


@app.route('/form')
def show_form():
    return render_template('greeter_form.html')

# 2-Grab the user input and define the function to use it in:


compliments = ['cool', 'clever', 'pythonic']


@app.route('/greet')
def greet():
    username = request.args['username']
    wants_compliments = request.args.get('wants_compliments')
    comps = sample(compliments, 2)
    return render_template('greet.html', username=username, wants_compliments=wants_compliments, compliments=comps)


@app.route('/lucky')
def show_lucky_num():
    """returns a random num"""
    num = randint(1, 3)
    msg = 'nice short number, isnt it?'
    return render_template('lucky_number.html', lucky_number=num, lucky_msg=msg)


@app.route('/spell/<word>')
def spell(word):
    return render_template('spell.html', word=word)


@app.route('/hello')
def say_hello():
    return render_template("hello.html")


@app.route('/bye')
def say_bye():
    return 'bye bye'

# QUERY PARAMS


@app.route('/search')
def search():
    term = request.args['term']
    sort = request.args['sort']
    print(request.args)
    return f"Searching for {term} sorting by {sort}"


# POST REQUESTS:
# RAW POST REQUEST:
@app.route('/post', methods=["POST"])
def post_demo():
    return "this is post request"


# MANAGE A FORM SUBMISSION:
# route and markup of the form:
@app.route('/add-comment')
def add_comment_form():
    """first set a get request to show the form to the user so he can input the data"""
    return """
    <form method="POST">
        <input type='text' placeholder='write comment here' name='comment'/>
        <input type='text' placeholder='enter username here' name='username'/>
        <button>Submit</button>
    </form>
    """

# route of the POST request:


@app.route('/add-comment', methods=["POST"])
def save_comment():
    """ grab the inputs by their name attributes, they're stored in 'request.form'"""
    user_comment = request.form["comment"]
    user_name = request.form["username"]
    msg = f"""
    Saved. Your comment:
    <ul>
        <li>text: "{user_comment}"</li>
        <li>User name: {user_name} </li>
    </ul>
    """
    return msg


# VARIABLES IN URLS:

USERS = {
    'rubia': 'the puddle',
    'astro': 'the sausage',
}


@app.route('/user/<username>')
def show_user_profile(username):
    """mind function parameter exactly the same as <decorator> argument"""
    name = username
    description = USERS[username]
    return f"""
    <ul>
        <li>Name: {name}</li>
        <li>Description: {description}</li>
    </ul>
    """


phrases_db = {
    1: 'flask is cool',
    2: 'python is nice',
    3: 'snow is a character'
}


@app.route('/phrases/<int:id>')
def find_post(id):
    """mind the 'int' syntax when using numbers in urls, will trigger the function
    only if there's an integer there, otherwise woun't run the view function"""
    # this method will return 'not here' by default if no phrase is found:
    phrase = phrases_db.get(id, 'phrase not here')
    return f"<p>{phrase}</p>"

# MULTIPLE VARIABLES IN URL:


@app.route('/r/<subreddit>')
def show_subreddit(subreddit):
    return f"Subreddit: {subreddit}"


@app.route('/r/<subreddit>/comments/<int:post_id>')
def show_comments(subreddit, post_id):
    """multiple url variables must match with multiple view func paramters"""
    return f"Comments :{subreddit}. From {subreddit}"

# JSONIFY
@app.route('/movies/json')
def get_movies_json():
    json_data = jsonify(list(movies))
    return json_data

```

Views are functions that return strings of html according to requests.
Routes are the subdirectories of the app

---

## redirect

## flash

Full syntax:

```python
from flask import Flask, request, render_template, redirect, flash, jsonify
from flask_debugtoolbar import DebugToolbarExtension

app = Flask(__name__)

# DEBUGGER:
# Cature redirection with debugger:
app.config['SECRET_KEY'] = "caca"
debug = DebugToolbarExtension(app)
# to stop debugger capture:
app.config['DEBUG_TB_INTERCEPT_REDIRECTS'] = False


# REDIRECT POST to GET:
# fake db:
movies = {'Amadeus', 'Chicken run', 'rambo'}


@app.route('/movies')
def show_all_movies():
    """show all movies in DB and prompt to add new movie"""
    return render_template('movies.html', movies=movies)


@app.route('/movies/new', methods=['POST'])
def add_movie():
    """adds new movie to DB and redirects to confirmation page, doesn't
    reuse '/movies/new' route template because that would cause the weird 'are you sure
    you want to resubmit the form' message and other weird behavoir, so it goes to /movies route,
    could also redirect to a confirmation of 'movie added' page and then go back to home"""
    title = request.form['title']
    # check if new movie exists and add to db:
    if title in movies:
        # flash to user with category
        flash('Movie already exists', 'error')
    else:
        movies.add(title)
        # FLASH TO USER with category:
        flash("Movie added!!", 'success')
    # redirect:
    return redirect('/movies')


# Redirect old to new:
@app.route('/old-page')
def redirect_to_home():
    return redirect('/home')


@app.route('/home')
def home_page():
    return render_template('home.html')


```

will be two requests: the first one with 302, the second one (made automatically by the browser) with code 200

## flash message

In app.py:

```python
flash('message-text!!', 'error/success/whatever')
```

In base.html template:

```html5
<body>
    <!-- check if there are flash messages, and run only if so -->
    {% with messages = get_flashed_messages(with_categories=true) %}
    {% if messages %}
    <section class="messages">
    {% for category, msg in messages %}
        <p class="{{category}}">{{msg}}</p>
    {%endfor%}
    </section>
    {% endif %}
    {%endwith%}
    <h1>All movies{%block title %} {%endblock%}</h1>
    {%block content%}
    {%endblock%}
</body>

```

In CSS, style .error, .success or whatever category names you assigned.

# redirecting

Redirecting to different endpoints. Used when logging in users, authenticating, etc.
To redirect:

- send HTTP response with status code 302 (or other "redirect code")
- must contain a URL for browser to re-request.

## jsonify

return json for apis, and also set the header of the request to JSON, to specify to all browser what type of data it contents. Mind sets can be JSON, have to be sets, lists or dictionaries.

```python
from flask import jsonify
# JSON
movies = {'rambo', 'rocky'}

@app.route('/movies/json')
def get_movies_json():
    json_data = jsonify(list(movies))
    return json_data
```

---

# WTForm

Four steps: 1-forms.py, 2-app.py, 3-form.html, 4-form testing:

1. forms.py:

```python
# Import class:
from flask_wtf import FlaskForm
# Import field types
from wtforms import StringField, FloatField, IntegerField, BooleanField, DateField, RadioField, SelectField
# Import validators:
from wtforms.validators import InputRequired, Optional, Email #(full list of validators in docs)


#Field types (static):
class AddSnackForm(FlaskForm):
    """Form for adding snacks."""

    name = StringField("Snack Name (required)", validators=[
                       InputRequired(message='Snack name cant be blank!')])
    email = StringField('Your Email (optional)',
                        validators=[Optional(), Email()])
    qtty = IntegerField('How many?')
    price = FloatField("Price in USD (required)", validators=[
                       InputRequired(message='please enter a valid number')])
    healthy = BooleanField('Healthy snack')  # return True/False
    best_by = DateField('Date of consumption')
    category = RadioField(
        'Category', choices=[('ic', 'Ice Cream'), ('ch', 'Chocolate'), ('candy', 'Candybar')])  # 'ic' will be sent as value, 'Ice Cream' is the label
    made_in = SelectField(
        'State', choices=[("ca", "California"), ('nv', 'Nevada'), ('ny', 'New York')])
    # Convert string to int from selectfield:
    priority = SelectField('Priority', choices=[
                           (1, 'high'), (2, 'low')], coerce=int)
    # List comprehension for select field:
    state = SelectField('State', choices=[(st, st) for st in states])


states = ['AL', 'AK', 'AR', 'PA']


#Dynamic SelectField:
#1. in forms.py
class AddEmployeeForm(FlaskForm):
    name = StringField('Full name')
    state = StringField('State')
    dept_code = SelectField('Department code')
```

2. form.html:

```markdown
<!-- OPTION 1: RENDER ALL FORM FIELDS IN SAME LOOP: -->

<h1>New snack</h1>
<!-- Mind that for the double purpose route there's no need to add an 'action' attr to the form, since it will automatically direct to the route where the form is called -->
<form method="POST">
    {{ form.hidden_tag() }}
    <!--Adds the hidden CSRF token-->
    {% for field in form if field.widget.input_type != 'hidden' %}
    <div class="form-group">
        <!-- mind 'class_' for styling purposes  -->
        {{field.label(class_='style-me')}}
        {{field(class_='style-me')}}
        <!-- show validation errors if any, with the label: -->
        {%for err in field.errors%}
        <small class="form-text">
            {{err(class_='style-this-message-please')}}
        </small>
        {%endfor%}
    </div>
    {% endfor %}
    <button>Add snack</button>
</form>

<!-- ---------------------------------------------------------------- -->
<!-- OPTION 2: RENDER FORM FIELDS INDIVIDUALLY ONE AT A TIME -->
<form>
    <!-- first the validation: -->
    {{ form.hidden_tag() }}
    <!-- then the fields: -->
    {{form.name.label(class_='red')}}
    {{form.name(class_='input-name')}}
    {{form.state.label(class_='brown')}}
    {{form.state(class_='input-state')}}
    {{form.dept_code.label(class_='blue')}}
    {{form.dept_code(class_='input-dept')}}
</form>
```

3. app.py:

> ALL NON GET ROUTES RETURN REDIRECT

```python
from forms import AddSnackForm
from flask import Flask, request, render_template, redirect

app = Flask(__name__)
# 2. Database config:
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql:///my_database'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SQLALCHEMY_ECHO'] = True
# Form and others:
app.config['SECRET_KEY'] = 'caca'

# 3. Call db:
connect_db(app)
# DUAL PURPOSE ROUTE: RENDERS THE FORM, VALIDATES THE FORM AND GRABS THE DATA FROM THE FORM:

# STATIC SELECT FIELD:
@app.route('/new-snack', methods=['GET', 'POST'])
def new_snack():
    """renders form, validates form and handle data"""

    form = AddSnackForm()

    # checks if is a post request, if CSRF token from form is valid, and runs validations:
    if form.validate_on_submit():
        # if it's a post request and token is valid: grab form's data, use it and redirect
        # to 'success' page or wherever:
        name = form.name.data
        price = form.price.data
        flash(f"Added {name} at {price}")
        return redirect("/home")
    # if it's not a post request, means that is a get request to just render
    # the form for the user to complete, so will only render it, or if the CSRF token is
    # invalid, will just render the form again:
    else:
        # access errors if you need them here, otherwise in template:
        errors = form.errors
        print(errors)  # ['Not a valid integer number', etc]
        return render_template(
            "snack_add_form.html", form=form)


# DYNAMIC SELECT FIELD:
@app.route('/emps/new', methods=['GET', 'POST'])
def add_employee():
    form = AddEmployeeForm()
    # DYNAMIC SELECT FIELD SETUP:
    depts = db.session.query(Department.dept_code, Department.dept_name)
    form.dept_code.choices = depts
    # Validation and grab data:
    if form.validate_on_submit():
        state = form.state.data
        name = form.name.data
        dept_code = form.dept_code.data
        # ADD AND COMMIT:
        new_emp = Employee(name=name, state=state, dept_code=dept_code)
        db.session.add(new_emp)
        db.session.commit()
        flash('Employee added. Welcome aboard')
        return redirect('/emps')
    else:
        return render_template('new-form.html', form=form)

# UPDATE FORM ROUTE
#show form with fields filled with existing data from db for the user to edit.
@app.route("/users/<int:u_id>/edit", methods=["GET", "POST"])
def edit_user(u_id):
    """Show user edit form and handle edit."""

    user = User.query.get_or_404(uid)
    form = UserForm(obj=user)

    if form.validate_on_submit():
        user.name = form.name.data
        user.email = form.email.data
        db.session.commit()
        flash(f"User {uid} updated!")
        return redirect(f"/users/{uid}/edit")

    else:
        return render_template("user_form.html", form=form)

```

4. tests.py:

```python

# Step 1, disable CSRF token for testing POST requests:
app.config['WTF_CSRF_ENABLED'] = False

# Step 2: Write tests:


class SnackViewsTestCase(TestCase):
    """Tests for views for Snacks."""

    def test_new_snack(self):
        with app.test_client() as client:
            resp = client.get("/add")
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertIn('<form id="snack-add-form"', html)

    def test_snack_add(self):
        with app.test_client() as client:
            d = {"name": "Test2", "price": 2}
            resp = client.post("/add", data=d, follow_redirects=True)
            html = resp.get_data(as_text=True)

            self.assertEqual(resp.status_code, 200)
            self.assertIn("Added Test2 at 2", html)

```

---

2. Install:
   - (check venv active)
   - `pip install flask-wtf`

This generic python library provides templating, validation and security for forms.
Flask-WTF is the specific integration for flask.
[flask-WTF](https://flask-wtf.readthedocs.io/en/stable/)

---

[field_types](https://wtforms.readthedocs.io/en/2.3.x/fields/#basic-fields)

CSRF (Cross Site Request Forgery) token: is a token that is sent with the form, but the user can't touch it because is hidden. With this token we make sure that the form received by our flask view is the same form coming from our served form template, with no hidden inputs nor anything that we didn't define. Prevents forging post requests to sites that we don't want to request. CSRF is checked by the server when submitted.

### form-session-flow

1.  Show form view:

    ```python
    @app.route('/form')
    def show_form():
        return render_template('form1.html')
    ```

2.  Form template:

    ```html
    {% extends 'base.html'%} {%block content%}

    <form action="/handle-form">
      Nickname?
      <input name="nickname" type="text" />
      Number?
      <input name="number" type="text" />
      <input type="submit" />
    </form>
    {%endblock%}
    ```

3.  Handle user input:
    (`request.args` for GET requests
    `request.form` for POST requests)

    ```python
    @app.route('/handle-form')
    def handle_form():
       session['nickname'] = request.args['nickname']
       session['number'] = int(request.args['number'])

       return render_template('confirmation.html',
                              # this is autmatically added by flask without us having to pass it:
                              # nickname=session['nickname'], number=session['number']
                              )
    ```

4.  Confirmation template:

    ```html
    {% extends 'base.html'%} {%block content%}

    <h2>done</h2>
    <ul>
      <li>Your nickname has been stored as {{session['nickname']}}</li>
      <li>Your number has been stored as {{session['number']}}</li>
    </ul>

    {%endblock%}
    ```

5.  Server Sessions, not flask default, to import it check docs about flask Sessions.

## forms

Grab data from forms:

```python
# from input name:
algo = request.form['algo']
# from multiple checkboxes:
muchos = request.getlist('muchos')
# from url or other stuff:
aver = request.get('aver')
```

`request.args` for GET requests
`request.form` for POST requests

```html
{% extends 'base.html'%} {%block content%}
<ul class="movies">
  {%for movie in movies%}
  <li>{{movie}}</li>
  {%endfor%}
</ul>
<form action="/movies/new" method="POST">
  <h2>Add your movie:</h2>
  <input type="text" placeholder="movie title" name="title" />
  <button>submit</button>
</form>
{%endblock%}
```

---

## authentication_and_authorization

## authentication

#### flask-Bcrypt

1. models.py:

#### models.py

```python
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt

db = SQLAlchemy()

bcrypt = Bcrypt()

def connect_db(app):
    db.app = app
    db.init_app(app)


class User(db.Model):
    """site user"""

    __tablename__ = "users"

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(db.Text, nullable=False, unique=True)
    password = db.Column(db.Text, nullable=False)

    @classmethod
    def register(cls, username, pwd):
        """Register user w/hashed password & return user."""

        hashed = bcrypt.generate_password_hash(pwd)
        # turn bytestring into normal string:
        hashed_utf8 = hashed.decode("utf8")

        # return instance of user w/username and hashed pwd
        return cls(username=username, password=hashed_utf8)

    @classmethod
    def authenticate(cls, username, pwd):
        """Validate that user exists & password is correct.
        Return user if valid; else return False.
        """
        u = User.query.filter_by(username=username).first()

        if u and bcrypt.check_password_hash(u.password, pwd):
            # return user instance
            return u
        else:
            return False


class Tweet(db.Model):
    __tablename__ = 'tweets'

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    text = db.Column(db.Text, nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'))

    user = db.relationship('User', backref="tweets")
```

2. app.py:

#### aapp.py

```python
from flask import Flask, render_template, redirect, session, flash
from flask_debugtoolbar import DebugToolbarExtension
from models import connect_db, db, User
from forms import RegisterForm, LoginForm
from sqlalchemy.exc import IntegrityError

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "postgres:///hashing_login"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
app.config["SQLALCHEMY_ECHO"] = True
app.config["SECRET_KEY"] = "abc123"

connect_db(app)
db.create_all()

toolbar = DebugToolbarExtension(app)

@app.route("/")
def homepage():
    """Show homepage with links to site areas."""

    return render_template("index.html")


@app.route("/register", methods=["GET", "POST"])
def register():
    """Register user: produce form & handle form submission."""

    form = RegisterForm()

    if form.validate_on_submit():
        name = form.username.data
        pwd = form.password.data

        #save encrypted pwd in db:
        user = User.register(name, pwd)
        db.session.add(user)
        try:
            db.session.commit()
        except IntegrityError:
            form.username.errors.append('Username taken, please pick another')
            return render_template('register.html', form=form)

        session["user_id"] = user.id
        flash('Welcome!', 'success')
        return redirect("/secret")
    else:
        return render_template("register.html", form=form)


# LOGIN USER AND KEEP HIM LOGGED IN:
@app.route("/login", methods=["GET", "POST"])
def login():
    """Produce login form or handle login."""

    form = LoginForm()

    if form.validate_on_submit():
        name = form.username.data
        pwd = form.password.data
        # authenticate will return a user or False
        user = User.authenticate(name, pwd)

        if user:
            session["user_id"] = user.id  # keep user logged in
            return redirect("/secret")
        else:
            form.username.errors = ["Bad name or password"]
    return render_template("login.html", form=form)

# CHECK IF USER'S LOGGED IN AND SHOW SECRET OR REDIRECT TO LOG IN PAGE:
@app.route("/secret")
def secret():
    """Example hidden page for logged-in users only."""

    if "user_id" not in session:
        flash("You must be logged in to view!")
        return redirect("/")
        # alternatively, can return HTTP Unauthorized status:
        #
        # from werkzeug.exceptions import Unauthorized
        # raise Unauthorized()
    else:
        return render_template("secret.html")


@app.route("/logout")
def logout():
    """Logs user out and redirects to homepage."""

    session.pop("user_id")
    flash("Goodbye!", "info")
    return redirect("/")


# SHOW ALL TWEETS  CREATE TWEET FORM FOR LOGGED IN USER
@app.route('/tweets', methods=['GET', 'POST'])
def show_tweets():
    if "user_id" not in session:
        flash("Please login first!", "danger")
        return redirect('/')
    form = TweetForm()
    all_tweets = Tweet.query.all()
    if form.validate_on_submit():
        text = form.text.data
        new_tweet = Tweet(text=text, user_id=session['user_id'])
        db.session.add(new_tweet)
        db.session.commit()
        flash('Tweet Created!', 'success')
        return redirect('/tweets')

    return render_template("tweets.html", form=form, tweets=all_tweets)

# DELETE TWEET IF LOGGED IN AND user is OWNER OF TWEET:
@app.route('/tweets/<int:id>', methods=["POST"])
def delete_tweet(id):
    """Delete tweet"""
    if 'user_id' not in session:
        flash("Please login first!", "danger")
        return redirect('/login')
    tweet = Tweet.query.get_or_404(id)
    if tweet.user_id == session['user_id']: # CHECK IF OWNER OF TWEET TO GIVE PERMISSION TO DELETE IT.
        db.session.delete(tweet)
        db.session.commit()
        flash("Tweet deleted!", "info")
        return redirect('/tweets')
    flash("You don't have permission to do that!", "danger")
    return redirect('/tweets')
```

3. templates:

#### templates

3.1 base.html

```markdown
<body>
  <nav class="navbar navbar-light bg-primary justify-content-between ">
    <!-- CHECK USER LOGGED IN -->
    <div class="container">
      <a href="/" class="navbar-brand text-light">Stupid Twitter</a>
      <ul class="nav navbar-nav flex-row float-right">
        {% if session['user_id'] %}
        <li class="nav-item">
          <a class="nav-link pr-3 text-light" href="/logout">Logout</a>
        </li>
        {% else %}
        <li class="nav-item">
          <a class="nav-link pr-3 text-light" href="/login">Login</a>
        </li>
        <li class="nav-item">
          <a class="nav-link text-light" href="/register">Sign up</a>
        </li>
        {% endif %}
      </ul>
    </div>
  </nav>

  <!-- FLASH MESSAGES -->
  <div class="container" style="margin-top: 2em">
    <!-- 'category' is the second arg passed for flash messages in views, then style by category name-->
    {% for category, msg in get_flashed_messages(with_categories=True) %}
    <div class="alert alert-{{category}}">{{ msg }}</div>
    {% endfor %}

    {% block content %}
    {% endblock %}

  </div>
</body>
```

4. forms.py:

#### forms.py

```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField
from wtforms.validators import InputRequired


class UserForm(FlaskForm):
    username = StringField("Username", validators=[InputRequired()])
    password = PasswordField("Password", validators=[InputRequired()])


class TweetForm(FlaskForm):
    text = StringField("Tweet Text", validators=[InputRequired()])
```

#### flask-bcrypt_basic_syntax

- `pip install flask-bcrypt`

```python
from flask_bcrypt import Bcrypt

# Instantiate:
bcrypt = Bcryp()

# Capture user's input and generate hashed salted password:
my_hash = bcrypt.generate_password_hash("boquita123")

# hash to store in db:
my_hash # b'$2b$12$s.tjeALK2I7rfI2gV27me.mkZu5IQd1Y1EBAXsbTvNExIEQcID/te'

# User login check:
bcrypt.check_password_hash(my_hash, "boquita123") #True
bcrypt.check_password_hash(my_hash, "boquita124") #False

```

Will use this algorithm to manage our passwords.

#### plain python bcrypt:

- `pip install bcrypt`

```python
import bcrypt
salt = bcrypt.gensalt(rounds=14) # will generate random salt every time its called.
salt # b'$2b$12$uYNRTDE7RrMvwDcF9f1Yyu' (string of bytes, binary)

users_password = b'monkey123' # get the pswrd from the user and convert it in binary string.
# call method to generate salted and hashed password:
bcrypt.hashpw(users_password, salt) # b'$2b$12$uYNRTDE7RrMvwDcF9f1Yyuvuu48PzANrWy88Iz3z1tRTfdXi6DlNW' (this is the string that we will store as user's password)
```

![bcrypt_hash_structure](/images/bcrypt.jpg)

### cryptographic hashing functions

The tiniest change in the input will provike a hughe change in the output, so it's impossible to reverse engineer it. We can't compare results. Popular cryptographic hashes:

- MD5, - SHA (used for cryptocurrencies). Popular password hashes: - Argon2, - Bcrypt, - Scrypt.

### hashing functions

Salt and hash: take the password, add random characters to it, and lastly hash it, so this will prevent repeated passwords and reverse engineering of the hash.
![popular_hashing_algorythms](/images/hash.jpg)

Built-in python hash function (no good for passwords): `import hash` `hash('something') #21312312312`
NEVER store passwords in plain text. Use a hash function (hash function is any function that can be used to map data of arbitrary size to fixed-size values. It's deterministic: from one value, you have to obtain always the same output from the same input). Cryptographic hash functions: input --- crypto hash function -- output. You can't deduct the input from the output. Hashing performs a one-way transformation of the password. Will hash the stored password and also what the user types, if both match, login, else not allowed.

AUTHENTICATION verify that a person is who he says he is, and sign in. (passwords, username, email, etc). NEVER store passwords in plain text. Use hashing.
AUTHORIZATION once you're verified, you have permissions to do things, and not to do others (you can create a post, but can't delete other people's posts). Moderators have other tier of permissions once authenticated. (if user, permission to do something, etc.)

---

## COOKIES

### set cookie from server side

    ```python
    from flask import Flask, make_response

    @app.route('/demo')
    def demo():
        content = "<h1>Hello</h1>"
        res = make_response(content)
        res.set_cookie('fav-color', 'blue')
        return res
    ```

### read cookies

      ```python
      from flask import Flask, request

      @app.before_request
      def print_cookies():
          print(request.cookies) # {'fav_color': 'blue', 'location': 'las totoras motel'}
      ```

### cookie options

      - Expiration
        - default is "as long as browser is running" (session cookie)
      - Domain (which domain should the cookie be sent to)
      - HttpOnly (not accessible via javascript)

    - Chrome: Dev Tools - Application - Storage - Cookies
    - Key-value pair, stored in the browser (client side). When making a request to a website, the browser sends all the cookies it has in storage (in the header of the request), when the server receives them, uses only the ones that can understand, and ignore the rest. If not, the server will instruct the browser what to do about storing cookies and how they will be structured. The client will send those cookies to the server always with every single request. (similar to JS local storage, with the difference that cookies are sent to server). with this implementation we now have state in our http protocol. The server can also send something in the http response instructing the browser to store a cookie, and the browser will store it in the client side.

7.  Types of browser storage:

    1.  LocalStorage
        - not sent to server
        - Stores data with no expiration date, and gets cleared only through JavaScript, or clearing the broser cache.
        - It's domain specific (can't use it accross different websites).
        - Storage limit is much larger than cookies.
        - For complex stuff or things we don't need to be in the server
    2.  SessionStorage (different than flask session)
        - not sent to server
        - Stores data only for until the browser tab is closed.
        - Storage limit larger than cookie.
    3.  Cookie
        - 4kb limit (must be light because:)
        - They are sent to servers
        - server and client can read them
        - Are made secure by setting the OnlyHttpOnly flag as true for that cookie. This prevent client-side access to it.
        - Sent from browser to server for every request to the same domain.
        - Set usually from the server side.
        - older browsers support
    4.  Flask-session.

## flask_sessions

Http is a stateless protocol, it doesn't "remember" nothing. An http request has no history, and is entirely separate from what came before or after.
If want to add things to a shopping cart, or stay authenticated in a website, must use cookies or sessions.

1. Browser Sessions
   Treat session just like a dictionary

   ```python
   from flask import Flask, session

   app = Flask(__name__)
   app.config['SECRET_KEY'] = "myPassword"

   @app.route('/some-route')
   def some_route():
       """flask takes the session's key-value pair, it will serialize it, will digitally sign it, and send the result as a cookie as part of the response that the client gets back. That cookie will then be stored in the browser and sent along with future requests."""
       # SET SESSION VALUES:
       session['username'] = 'coriolano32'
       session['tools'] = ['hammer', 'saw', 'screw driver']
       return 'Ok'

       # UPDATE SESSION VALUES:
       session['username'] = 'pepito123'

       #ON THE SERVER SIDE, TO READ A SESSION, JUST ACCESS IT LIKE IN A DICT:
       session['username'] # 'pepito123'
       session['tools'] # ['hammer', 'saw', 'screw driver']
   ```

   Session specs:

   - contain info for the current browser
   - Preserve data type (lists stays lists, etc)
   - Are cryptographically signed, user's can't modify data. ("signed" means that is codified by an algorithm in the server)
     It's a dictionary that manage cookie creation, reading and sending. It's more secure than cookies.

---

# jinja_templates

1. index.html:

```markdown
{% extends 'base.html' %}

{% block content %}

<h1>This is index page!!!</h1>

{% endblock %}
```

2. base.html:

```markdown
<html>
<head>
  <link rel="stylesheet" href="https://bootswatch.com/4/flatly/bootstrap.min.css">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.12.0/css/all.min.css">
</head>

<body>
  <nav class="navbar navbar-light bg-primary justify-content-between ">
    <!-- CHECK USER LOGGED IN -->
    <div class="container">
      <a href="/" class="navbar-brand text-light">Stupid Twitter</a>
      <ul class="nav navbar-nav flex-row float-right">
        {% if session['user_id'] %}
        <li class="nav-item">
          <a class="nav-link pr-3 text-light" href="/logout">Logout</a>
        </li>
        {% else %}
        <li class="nav-item">
          <a class="nav-link pr-3 text-light" href="/login">Login</a>
        </li>
        <li class="nav-item">
          <a class="nav-link text-light" href="/register">Sign up</a>
        </li>
        {% endif %}
      </ul>
    </div>
  </nav>

  <!-- FLASH MESSAGES -->
  <div class="container" style="margin-top: 2em">
    {% for category, msg in get_flashed_messages(with_categories=True) %}
    <div class="alert alert-{{category}}">{{ msg }}</div>
    {% endfor %}

    {% block content %}
    {% endblock %}

  </div>

</body>
</html>
```

3. further-htmls:

## jinja_wtform

```markdown
{% extends 'base.html'  %}

{% block content %}

<h1 class="display-1">Login</h1>
<p class="lead">Login Below. Don't have an account? <a href="/register">Register here</a></p>

<form method="POST">

{{ form.hidden_tag() }}

{% for field in form
    if field.widget.input_type != 'hidden' %}

  <p>
    {{ field.label }}
    {{ field(class_="form-control") }}

    {% for error in field.errors %}
    <span class="form-text text-danger">{{ error }}</span>
    {% endfor %}

  </p>
  {% endfor %}

<button class="btn btn-success" type="submit">Login</button>

</form>
{% endblock %}
```

## jinja_loops

```html
<body>
  {% for char in word %}
  <h3>{{char}}</h3>
  {% endfor%}
</body>
```

## jinja_if

In a same template, add different content according to different conditions (user loged in or not, etc)

```markdown
<ul>
  {% if (posts|length > 0)%} 
  {% for post in posts%}
  <li>{{post.title}}</li>
  {%endfor%} 
  {%else%}
  <p>No posts yet</p>
  {%endif%}
</ul>
{% if number == 2 %}
<h2>That's extra cool!!</h2>
{% else %}
<h2>I don't like that number {{number}}</h2>
{% endif %}
```

## dynamic variables

```html
Jinja will replace {{msg}} with the value of msg passed when rendering.
```

# jinja setup

1. create template directory, in the same directory that the app.py file. This is the directory model:

   - my_project_dir/
     - venv/
     - app.py
     - templates/
       - hello.html
     - static
       - my-css.css (in base.htm: `<link rel="stylesheet" href="/static/my-css.css">`)
       - my-script.js

2. Create hml files inside the templates directory
3. import `render_template` to app.py

Templates allows to return dynamic html, embeding variables and other stuff.
Jinja comes with flask, no need to install it.

---

# API

## server_side_requests(flask)

Make requests from the server side
Some APIs have an SDK (Software Development Kit), oriented to the use of the API in ios, android, python or JS, etc.

app.py:

```python

import requests
from keys import key
from flask import Flask, render_template, request, redirect
from forms import LocationForm

app = Flask(__name__)

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
app.config['SQLALCHEMY_ECHO'] = True
app.config['SECRET_KEY'] = 'caca'


API_BASE_URL = 'http://www.mapquestapi.com/geocoding/v1/address'


def get_coordenates(address):
    response = requests.get(API_BASE_URL,
                            params={'key': key, 'location': address})
    data = response.json()
    lat = data['results'][0]['locations'][0]['latLng']['lat']
    long = data['results'][0]['locations'][0]['latLng']['lng']
    coords = {'lat': lat, 'lng': long}
    return coords


@app.route('/', methods=['GET', 'POST'])
def get_location():
    form = LocationForm()
    if form.validate_on_submit():
        address = form.address.data
        coords = get_coordenates(address)
        return redirect(f"/result/{coords}")
    else:
        return render_template('location-form.html', form=form)

```

## python requests

No flask, no server, just python

1. check venv active
2. `pip install requests`

```python
import requests

# GET REQUESTS:

resp1 = requests.get("https://itunes.apple.com/search", params={"term": "billy bragg", "limit": 3})
resp1.text
resp1.status_code
resp1.json() #converts JSON response to Python dict

data = resp1.json()

for result in data['results']:
print(result['trackName'])

# --------------------------------------------

# POST REQUESTS:
# Option 1, with json
pref = request.post('https://endpoint', json={'key':'value value', 'other': 2})

mydict = {
    'username': 'chicken',
    'tweets': [
        'hello', 'goodbye', 'my god'
    ]
}
req1 = request.post('https://endpoint', json=mydict) # library will convert python dict into json

# Option 2, with data:
ver = request.post('https://endpoint', data={'key':'value', 'other': 2})
```

## API keys

### secret_keys

1. .gitignore:

```
secrets.py
venv/
```

2. secrets.py:

```python
API_SECRET_KEY = 'fdsfsdf234234234234'
```

3. app.py:

```python
from secrets import API_SECRET_KEY
```

4. Warn other developers to make their own secret key.

HTTP requests can be:

1. - Client side requests (via AJAX), are done from the browser. Some api's don't allow this.
2. - Server side requests:
     ![graphic](/images/server-requests.jpg)
     can manage passwords, privacy, keys and requests that are not admitted from browsers.

# dir

folders structure:

- my_project_dir/
  - venv/
  - app.py
  - templates/
    - hello.html
  - static
    - my-css.css (in base.htm: `<link rel="stylesheet" href="/static/my-css.css">`)
    - my-script.js

---

# setup

# setup flask server

1. Installing Flask
   1. mkdir or cd to directory
   2. create virtual env (`python3 -m venv venv`)
   3. activate venv (`source venv/bin/activate`)
   4. `pip install flask`
2. Make a python file to run your code:
   1. cd to project main folder
   2. `touch app.py`
   3. in 'app.py', instantiate your app:
      - `from flask import Flask`
      - `app = Flask(__name__)`
3. Start server:
   1. Production mode
      - `flask run` (with virtual env active)
      - `ctrl + C` stop server
   2. Development mode (debugger and restart activated):
      - `FLASK_ENV=development flask run`
      - (Set dev mode to default in local venv and not permanently:
        1. check venv active
        2. `export FLASK_ENV=development`
        3. `flask run` will run dev mode by default, until terminal window is closed.)
      - (change bash profile so it runs flask dev mode by default in all terminals):
        1. go to local .zshrc file
        2. paste `export FLASK_ENV=development`
        3. reboot all terminals
        4. `flask run` in venv will run in dev mode by default.
4. (If flask app file isn't called 'app.py':
   - `FLASK_APP=my_custom_name.py flask run` (mind the not spaced chars))

# Frameworks

Like a library, but more opinionated. They provide all the functions, classes and logic to manage the requests and responses of our web apps and programs.
"Which request to respond and how to respond."
Basically, web applications do this:

- handle web requests
- producte dynamic html based on those requests
- handle forms
- handle cookies
- connecto to databases
- provide user authentication
- cache pages for performance

# debug

## flask debug toolbar

Install:

1. check venv active
2. `pip install flask-debugtoolbar`
3. in app.py:

   ```python
   from flask_debugtoolbar import DebugToolbarExtension

   app = Flask(__name__)

   # config debugger:
   app.config['SECRET_KEY'] = "caca"
   debug = DebugToolbarExtension(app)
   # to stop debugger:
   app.config['DEBUG_TB_INTERCEPT_REDIRECTS'] = False
   ```

4. only works on pages where templates are returned.

## Python debugger pdb

```python
import pdb
pdb.set_trace()
#execution will stop and terminal will be ready to find type variables and find its values.
```

Key commands for pdb:
| key|command
| ? | Get help
| l | List code where I am
| p | Print this expression
| pp | Pretty print this expression
| n |Go to next line (step over)
| s |Step into function call
| r |Return from function call
| c |Continue to next breakpoint
| w |Print “frame” (where am I?)
| q |Quit debugger