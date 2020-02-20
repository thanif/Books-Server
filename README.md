# Books-Server
Create a web application with python + Flask + PostgreSQL and deploy on Heroku

# Install Postgre SQL
Following : https://tecadmin.net/install-postgresql-server-on-ubuntu/

# Step 1 – Enable PostgreSQL Apt Repository

sudo apt-get install wget ca-certificates

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'

# Step 2 – Install PostgreSQL on Ubuntu

sudo apt-get update

sudo apt-get install postgresql postgresql-contrib

# Step 3 – Create User for PostgreSQL

Now configure PostgreSQL to make is accessible by your normal users. Change your_username with your actual user already created on your Ubuntu system.

postgres-# CREATE ROLE your_username WITH LOGIN CREATEDB ENCRYPTED PASSWORD 'your_password';

postgres-# \q

Then switch to the user account and run createdb command followed by the database name. This will create a database on PostgreSQL.

su - your_username

createdb my_db // repalce my_db with the username

After that connect to the PostgreSQL server. You will be logged in and get database prompt. To list all available databases use these commands.

psql

talha=> \list

To disconnect from PostgreSQL database command prompt just type below command and press enter. It will return you back to the Ubuntu command prompt.

\q

# Install Heroku CLI

sudo snap install --classic heroku

After installing Heroku CLI using any method above, Verify installation by

heroku --version

Create a Heroku account if you have not one already and login to your Heroku account in CLI by

heroku login

# Create python virtual environment for the project

To create virtual environments we need virtualenv package. Install python virtualenv package using,

pip install virtualenv

Then create a new directory for the project. Let’s say books_server

mkdir books_server

cd books_server

Create a virtual environment named env inside the created directory by,

virtualenv env

This will create the virtual environment named env inside the books_server.

To activate this environment use this command inside books_server directory.

source env/bin/activate

# Create a sample code with Flask to check

For using Flask, first you need to install Flask. (Make sure that you have activated the virtual environment)

pip install Flask

Now create a file named app.py in books_server directory and put below code to test Flask before we move into real application development

from flask import Flask, request

app = Flask(__name__)

@app.route("/")

def hello():

    return "Hello World!"

@app.route("/name/<name>")

def get_book_name(name):

    return "name : {}".format(name)

@app.route("/details")

def get_book_details():
    
    author=request.args.get('author')
    
    published=request.args.get('published')
    return "Author : {}, Published: {}".format(author,published)

if __name__ == '__main__':
    
    app.run()

To execute above code run

python app.py

You can check the deployed server on http://127.0.0.1:5000/

From here let’s move to create our book details storing application.

# Create database

First create the database we need here for our application named books_store

sudo -u name_of_user createdb books_store

Now you can check the created database with,

psql -U name_of_user -d books_store

You should log into books_store data base if above command was success.

# Create configurations

We need to define configurations for deploying environments. create a file named config.py with below code.

import os

basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    
    DEBUG = False
    TESTING = False
    CSRF_ENABLED = True
    SECRET_KEY = 'this-really-needs-to-be-changed'
    SQLALCHEMY_DATABASE_URI = os.environ['DATABASE_URL']


class ProductionConfig(Config):
    
    DEBUG = False


class StagingConfig(Config):
    
    DEVELOPMENT = True
    DEBUG = True


class DevelopmentConfig(Config):
    
    DEVELOPMENT = True
    DEBUG = True


class TestingConfig(Config):
    
    TESTING = True

According to created configurations set “APP_SETTINGS” environment variable by running this in the terminal

export APP_SETTINGS="config.DevelopmentConfig"

Also add “DATABASE_URL” to environment variables. In this case our database URL is based on the created database. So, export 
the environment variable by this command in the terminal,

export DATABASE_URL="postgresql://localhost/books_store"

It should be returned when you execute echo $DATABASE_URL in terminal.

So, now our python application can get database URL for the application from the environment variable which is 

“DATABASE_URL”

Also put these 2 environment variables into a file called .env

export APP_SETTINGS="config.DevelopmentConfig"

export DATABASE_URL="postgresql://localhost/books_store"

# Database migration

Now we need to use flask_sqlalchemy package for database manipulations. Install it by,

pip install flask_sqlalchemy

And in app.py create db variable using SQLAlchemy using :

Also set app.config like below.

from flask import Flask, request

from flask_sqlalchemy import SQLAlchemy

Import os

app = Flask(__name__)

app.config.from_object(os.environ['APP_SETTINGS'])

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

from models import Book

@app.route("/")

def hello():
    
    return "Hello World!"
 
defined db=SQLAlchemy(app) will be used to handle the database transactions.

Here we had to import Book from models. models.py is described below.

Create a file named models.py and there we define our tables. Here we define our table as books and we create the model class as Book.

from app import db

class Book(db.Model):
    
    __tablename__ = 'books'

    id = db.Column(db.Integer, primary_key=True)
    
    name = db.Column(db.String())
    
    author = db.Column(db.String())
    
    published = db.Column(db.String())

    def __init__(self, name, author, published):
        
	self.name = name
        
	self.author = author
        
	self.published = published

    def __repr__(self):
        
	return '<id {}>'.format(self.id)
    
    def serialize(self):
        
	return {
            
	    'id': self.id, 
            
	    'name': self.name,
            
	    'author': self.author,
            
	    'published':self.published
        }

We have defined books table with the columns [id, name, author, published]

another requirement for database migration is manage.py file. Create a file named manage.py

from flask_script import Manager

from flask_migrate import Migrate, MigrateCommand

from app import app, db

migrate = Migrate(app, db)

manager = Manager(app)

manager.add_command('db', MigrateCommand)


if __name__ == '__main__':
    
    manager.run()

In manage.py file we use 2 more packages flask_script, flask_migrate.

Also we need psycopg2-binary package. Install them by,

pip install flask_script

pip install flask_migrate

pip install psycopg2-binary

Now we can start migrating database. First run,

python manage.py db init

Change connection string to export DATABASE_URL="postgresql:///books_store"

This will create a folder named migrations in our project folder. To migrate using these created files, run

python manage.py db migrate

Now apply the migrations to the database using

python manage.py db upgrade


psql -U talha -d books_store

This will create the table books in books_store database. you can check it in PostgreSQL command line inside books_store 
database by \dt command.

You can check the columns of the books table by \d books command.

# Finish the code

At this moment we have created the required database configurations and database. So now we can focus on database transactions in our python server. Here we will focus on app.py

import os

from flask import Flask, request, jsonify

from flask_sqlalchemy import SQLAlchemy
 
app = Flask(__name__)
 
app.config.from_object(os.environ['APP_SETTINGS'])

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
 
from models import Book
 
@app.route("/")

def hello():
    
    return "Hello World!"
 
@app.route("/add")

def add_book():
    
    name=request.args.get('name')
    
    author=request.args.get('author')
    
    published=request.args.get('published')
    
    try:
        
	book=Book(
            
	    name=name,
            
	    author=author,
            
	    published=published
        )
        db.session.add(book)
        db.session.commit()
        return "Book added. book id={}".format(book.id)
    except Exception as e:
	    return(str(e))
 
@app.route("/getall")

def get_all():
    
    try:
        books=Book.query.all()
        return  jsonify([e.serialize() for e in books])
    except Exception as e:
	    return(str(e))
 
@app.route("/get/<id_>")

def get_by_id(id_):
    
    try:
        book=Book.query.filter_by(id=id_).first()
        return jsonify(book.serialize())
    except Exception as e:
	    return(str(e))
 
if __name__ == '__main__':
    
    app.run()

We have used jsonify for make our response in JSON format. You can see that serialize method we created in Book class in models.py is used here to provide book objects as serialized.

Also, as now we have created manage.py now we can run our server locally by,
python manage.py runserver

And this will automatically refresh the local server with code changes.

http://127.0.0.1:5000/add?name=Twilight&author=Stephenie Meyer&published=2006 will add the book “Twilight” to the database.

http://127.0.0.1:5000/get/1 will return the book details of the book which id is 1 as a JSON.

http://127.0.0.1:5000/getall will return every book that have been stored our database.

At this moment database + python server is complete for our project as a REST API. But if you need to handle html files with this application we can further improve it by adding a folder named templates to our project root directory and add html files there.

# Get the server re-running after the setup

cd books_server

source env/bin/activate

export APP_SETTINGS="config.DevelopmentConfig"

export DATABASE_URL="postgresql:///books_store"

python manage.py runservers

Let’s create getdata.html with a form and note that the form method is POST.

I have used Bootstrap to make it a smooth web page.

<!DOCTYPE html>

<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
        crossorigin="anonymous">
</head>
 
<body>
    <div class="container">
 
        <div class="container">
            <br>
            <br>
 
            <div class="row align-items-center justify-content-center">
                <h1>Add a book</h1>
            </div>
            <br>
 
            <form method="POST">
 
                <label for="name">Book Name</label>
                <div class="form-row">
                    <input class="form-control" type="text" placeholder="Name of Book" id="name" name="name">
                </div>
                <br>
                <div class="form-row">
                    <label for="author">Author</label>
                    <input class="form-control" type="text" placeholder="Author Name" id="author" name="author">
                </div>
                <br>
                <div class="form-row ">
                    <label for="published">Published</label>
                    <input class="form-control " type="date" placeholder="Published" id="published" name="published">
                </div>
 
                <br>
                <button type="submit " class="btn btn-primary " style="float:right ">Submit</button>
            
            </form>
            <br><br>
        </div>
    </div>
 
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js " integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo "
        crossorigin="anonymous "></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js " integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49 "
        crossorigin="anonymous "></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js " integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy "
        crossorigin="anonymous "></script>
</body>
 
</html>

Now create a method to render this getdata.html through our python app.

import os

from flask import Flask, request, jsonify, render_template

from flask_sqlalchemy import SQLAlchemy
 
 
app = Flask(__name__)
 
app.config.from_object(os.environ['APP_SETTINGS'])

app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
 
from models import Book
 
@app.route("/")

def hello():
    
    return "Hello World!"
 
@app.route("/add")

def add_book():
    
    name=request.args.get('name')
    author=request.args.get('author')
    published=request.args.get('published')
    try:
        book=Book(
            name=name,
            author=author,
            published=published
        )
        db.session.add(book)
        db.session.commit()
        return "Book added. book id={}".format(book.id)
    except Exception as e:
	    return(str(e))
 
@app.route("/getall")

def get_all():
    
    try:
        books=Book.query.all()
        return  jsonify([e.serialize() for e in books])
    except Exception as e:
	    return(str(e))
 
@app.route("/get/<id_>")

def get_by_id(id_):
    
    try:
        book=Book.query.filter_by(id=id_).first()
        return jsonify(book.serialize())
    except Exception as e:
	    return(str(e))
 
@app.route("/add/form",methods=['GET', 'POST'])

def add_book_form():
    
    if request.method == 'POST':
        name=request.form.get('name')
        author=request.form.get('author')
        published=request.form.get('published')
        try:
            book=Book(
                name=name,
                author=author,
                published=published
            )
            db.session.add(book)
            db.session.commit()
            return "Book added. book id={}".format(book.id)
        except Exception as e:
            return(str(e))
    return render_template("getdata.html")
 
if __name__ == '__main__':
    
    app.run()

Here we have used render_template method and note that render_template has been imported from flask. http://127.0.0.1:5000/add/form will return this form.

# Commit changes using git and push to Heroku

We need to use gunicorn package here for Heroku. Either we don’t need to use gunicorn in our local machine, as we can add every package we installed in our project to a file using pip freeze command let’s install gunicorn to our project so we can add it to required packages list for Heroku.

pip install gunicorn

At the moment we have installed all the required packages for this simple project. We need to specify these packages in 
requirements.txt to identify and install when we push our project to Heroku. It can be simply done by,

pip freeze > requirements.txt

Above command will create the file requirements.txt in project root directory and add each package we have installed in our virtual environment.

Now we need to create Procfile to specify our application to Heroku.

web: gunicorn app:app

Also to specify which python version you need to use, create runtime.txt

python-3.6.5
 
Now to push our project into Heroku we should initialize a git repository for the project. Create an git repository by,

git init

And create a file named .gitignore in the project root directory.

.env

__pycache__/

env/

.gitignore

Now add project files to git and commit.

git add .

git commit -m "initial commit"

Now it is time to create an application on Heroku and push our code. To create an application according to your project use the below command.

heroku create name_of_your_application

This name should be a unique one. So you may have to try several times if the name has already taken by someone. Also if you have 5 applications already running on Heroku at the moment you cannot create another application with the free account. 

Either you have to delete an existing application from Heroku or upgrade your account.

Here I created the application named first-books-example

The link here in green color is the URL for our application and yellow color link is the git remote link on Heroku.

Now we can add this git remote link to our local git repository. Let’s name our remote as prod for the meaning of production.

In my case it is ,

git remote add prod https://git.heroku.com/first-books-example.git

Now set the configurations for Heroku application through Heroku CLI.

First set “APP_SETTINGS” environment variable.

heroku config:set APP_SETTINGS=config.ProductionConfig --remote prod

Then add Postgres addon to Heroku server by,
heroku addons:create heroku-postgresql:hobby-dev --app name_of_your_application

You can check variables you set for Heroku server by,

heroku config --app name_of_your_application

Now it is time to push our code into Heroku. It is done by using git.

git push prod master

This will install required packages and deploy your application on Heroku. You can check deployed application through the URL Heroku provided. But still there is one more thing to do. You cannot add or get data through database. That’s should happen because we still didn’t migrate the database in Heroku.

Run migrations by,

heroku run python manage.py db upgrade --app name_of_your_application

That’s all for this tutorial.

Deployed project is here. https://first-books-example.herokuapp.com/



# References

https://tecadmin.net/install-postgresql-server-on-ubuntu/

https://medium.com/@dushan14/create-a-web-application-with-python-flask-postgresql-and-deploy-on-heroku-243d548335cc
