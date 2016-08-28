---
layout: post
title: "Flask n' iOS Part 3: Data Persistence with PostgreSQL"
author: Trevor Senior
date: 2016-08-26
updated: 2016-08-26
headerImg: ""
headingTextColor: "#FEFEFE"
categories: flask ios postgresql
---


# Data Persistence with PostgreSQL

### Post Contents

###### 1. Overview

###### 2. Setting Up PostgreSQL 

###### 3. Adding a Model

###### 4. Making Moves

#### 1. Overview

Last time on Flask n' iOS, we built an endpoint, and we added some logic to store data from incoming POST requests. This is legitimately pretty damn cool. However, when we shut down the server, what happens to our ```data``` dictionary? The problem is that this isn't persistent, because it's stored in memory alongside the running flask process. When the process is killed, the ```data``` dictionary is lost. We can fix this with a database, which will securely store our data on disk for later retrieval. For the purpose of this tutorial, we'll use postgreSQL with SQLAlchemy, the database and a library for working with SQL databases, respectively. PostgreSQL is an efficient database that isn't too difficult to setup and use, which is why we're using it over sqlite or mysql. Enough talk, let's get our hands dirty!

###### *WARNING* be sure to remember to restart your server and activate your virtual environment whenever you're making changes to flask that you want nginx and gunicorn to reflect.

#### 2. Setting Up PostgreSQL

Our first task is to.... You guessed it, install more dependencies! Let's begin with postgreSQL:

```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

Upon installation, postgres will create a new user named 'postgres' on your machine. This is because postgres defines privledges for users who have been registered to use it. Thus, the default user upon installation with elevated privledges is 'postgres'. Additionally, postgresql has a special name for these privledged users, 'roles'. The bottom line is that we need to tell postgresql that our user, sammy, should have elevated privledges to create a database. In other words, sammy should have a role in postgres (yes, the 'role' terminology is confusing even to me). Let's do that now:

```
sudo -u postgres createuser --interactive
```

```
Enter name of role to add: sammy
Shall the new role be a superuser? (y/n) y
```

One more thing must be done to set up our postgres database. Creating the database! Now that we have an authenticated role/user, let's create a database:

```
sudo -u postgres createdb sammy
```

Traditionally, instead of creating a database and user/role named sammy, you would do something like '<your project name here>' instead. However, for simplicities sake, let's get through this part quickly. If you would like to do a deeper dive, check out one of Digital Ocean's excellent tutorials: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04


#### 3. Adding a Model

Now that postgres is good to go, let's switch gears back to flask development. If you don't have it activated already, activate your virtual environment for myproject, so that we can pip install some dependencies. Now let's install the following:

```
sudo apt-get install libpq-dev python-dev
these installations are used to solve a compatibility issue I found happen for ubuntu 16.06
```

```
pip install -U psycopg2
pip install Flask-SQLAlchemy
```

OK, that's all we'll need. As mentioned before, sqlalchemy is a python library for working with sql databases. Flask-SQLAlchemy is a thin wrapper on top which adds some API functionality to manage a flask app. You'll learn the same things by using SQLAlchemy or Flask-SQLAlchemy.

Now, let's add some sqlalchemy logic to our existing ```myproject.py``` flask app file.

```
from flask_sqlalchemy import SQLAlchemy
app.config['SQLALCHEMY_DATABASE_URI'] = "postgresql:///sammy"
db = SQLAlchemy(app)
```
The first couple lines are telling the app.config dictionary to have a key value pair that will later tell sqlalchemy where on the ubuntu filesystem our postgres database is. In this case, the full path name to the sammy database isn't used, because sqlalchemy can figure it out based on the default installation for postgreSQL. We then instantiate the SQLAlchemy module with a parameter of our app and we'll call that ```db``` from now on. The cool thing is that we can now use the sqlalchemy API here: http://flask-sqlalchemy.pocoo.org/2.1/ to effectively modify our database however we like. To that end, let's define a model. This model will help us create a database representation of storing URL's that are POST'ed to our server.

```
class Link(db.Model):
    __tablename__ = "links"
    id = db.Column(db.Integer, primary_key=True)
    url = db.Column(db.Text)

    def __init__(self, url):
        self.url = url

    def __repr__(self):
        return '<link %r>' % self.url
```

This may look confusing, but it's actually pretty simple. We create a python class called Link and extend the ```db.Model``` class. Then we define a special variable named ```__tablename__``` which will ultimately be what our table is called in our sammy database. After that, we define some columns in our database. ```id``` is a unique identifier for row entries in our table, whereas ```url``` is just a blob of text. SQLAlchemy API defines a bunch of other column types you can use for different models, Boolean for instance. Later on, if we have a ```Link``` object, we can do ```.url``` to access that property of the object just like any python object. Now that we have this model defined, let's use our ```db``` module to actually create the ```links``` table:

```
python
from myproject import db
db.create_all()
```

Now if we access our database via ```psql``` command, we would be able to enter ```\dt``` and it would display the current tables in the ```sammy``` database, one of which is ```links```.

#### 4. Making Moves

If you've made it this far without errors, take a moment to congratulate yourself. Ok, now back to work. The finale of this series begins now! We'll be using our ```db``` object from ```SQLAlchemy``` within a flask route for links of the interwebz. Let's see this in action with the following code:

```
@app.route('/links', methods=['POST', 'GET'])
def names():
        if request.method == 'GET':
                urls = [str(link.url) for link in Link.query.all()]
                return jsonify(urls)

        if request.method == 'POST':
                req_json = request.get_json()
                if req_json is not None and 'url' in req_json:
                        link = Link(req_json['url'])
                        db.session.add(link)
                        db.session.commit()

                        urls = [str(link.url) for link in Link.query.all()]
                        return jsonify(urls)
                return ('', 400)
```
(note: this code is vulnerable to an SQL injection, why? Also, how would you fix it?)

Ok, so what's the lowdown of this file? First, we create a new route that can handle POST and GET requests. EASY! Then, when a client makes a GET request, we do some interesting stuff. The ```Link``` object which extends ```db.Model``` has an attribute ```query``` which allows you to make queries against your database for the ```links``` table. In this case ```query.all()``` returns all the rows of the table. If you look at the ```SQLAlchemy``` API you can find some cool queries to filter based on parameters of your choice. Once we have a list of all the rows from the ```links``` table, we convert them from unicode to a regular string and serve them in a JSON array. SIMPLE!

Let's look at part number 2, the POST request handler. In this handler, we make sure the client gave us JSON with a ```url``` field, and then we instantiate a new ```Link``` object with the request's url. Now we do something very cool. We use the ```db.session``` attribute. The ```session``` attribute allows us to add multiple commands to it. In this instance, we're queueing up a ```.add(link)``` command. Once you have queued up all the commands that you would like to apply to your database, you call a method, ```commit()```. This is an important aspect of databases to understand. Essentially, you want all the commands that you queued up in the ```session``` to be applied to your database in unison once you call ```commit()```. It's important, because it allows the database to either complete all the commands, or fail at all the commands as well, rather than getting through half of them and then having the server die, and then you not knowing what has been committed to your database. If you would like to learn more about this, I highly recommend this coursera course: https://www.coursera.org/learn/cloud-computing-2 which describes the importance of maintaining atomic transactions.


Too much reading? Go ahead and test out your server with postman! If something isn't working, try just running

```
python myproject.py
```

and visiting your server on whichever port it's configured to go to. Once there, test out the ```/links``` route to see if GET works with no syntax errors.



### Wrapping Up

In this workshop series we learned how to set up our server infrastructure on digital ocean. We learned how to create a flask endpoint and test it with postman. Finally, we learned how to atomically use persistent data. What we did not learn was how to do this securely. If you followed this tutorial, someone can access an unprotected port on your server, AND they can cause an SQL injection. Most likely they can do some pretty nasty stuff to nginx as well. Likewise, we didn't learn how to automate our server restarts, and we didn't learn how to update our database through migrations (alembic). Again, this workshop was for beginners, so that you can see the different serverside communication components and how they interact.
