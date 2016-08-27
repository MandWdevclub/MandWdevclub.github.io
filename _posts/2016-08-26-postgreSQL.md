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


#Data Persistence with PostgreSQL

###Post Contents
######1. Overview
######2. Setting Up PostgreSQL 
######3. Adding a model
######4. Making Moves

####1. Overview

Last time on Flask n' iOS, we built an endpoint, and we added some logic to store data from incoming POST requests. This is legitimately pretty damn cool. However, when we shut down the server, what happens to our ```data``` dictionary? The problem is that this isn't persistent, because it's stored in memory alongside the running flask process. We can fix this with a database, which will securely store our data on disk for later retrieval. For the purpose of this tutorial, we'll use postgreSQL with SQLAlchemy, the database and a library for working with it respectively. PostgreSQL is an efficient database that isn't too difficult to setup and use, which is why we're using it over sqlite or mysql. Enough talk, let's get our hands dirty!

####2. Setting Up PostgreSQL


install postgreSQL

install pip packages (and the fix for psycopg2) http://stackoverflow.com/questions/28253681/you-need-to-install-postgresql-server-dev-x-y-for-building-a-server-side-extensi
sudo apt-get install libpq-dev python-dev


create user and db https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04

create model in flask app
first run create_all() before using the db (possibly automate this)
add a test row to your database  http://flask-sqlalchemy.pocoo.org/2.1/quickstart/

possibly add alembic migration

in GET route, display jsonified table
in POST route commit a new link



```
from flask import Flask, jsonify
app = Flask(__name__)

from flask_sqlalchemy import SQLAlchemy
app.config['SQLALCHEMY_DATABASE_URI'] = "postgresql:///sammy"


db = SQLAlchemy(app)
class User(db.Model):
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True)

    def __init__(self, email):
        self.email = email

    def __repr__(self):
        return '<E-mail %r>' % self.email



@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

