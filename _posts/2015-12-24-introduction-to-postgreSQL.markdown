---
layout: post
title:  "Introduction to PostgreSQL (Mac OS X)"
date:   2015-12-24 14:35:29 -0400
categories: tutorials
---
##Overview
This is an introduction intended to get a Mac OS X user off the ground and running locally with PostgreSQL.  Included is how to install postgres, connect to the psql]
q interactive terminal, set up a new user/password, create and delete databases, as well as how to create, update, and delete tables and table content.

##Getting Started
Let's download [PostgreSQL](http://www.postgresql.org/download/macosx/) on your machine.  

[__Mac OS X:__](http://www.postgresql.org/download/macosx/): I highly suggest installing [homebrew](http://brew.sh/) for current and future use:
{% highlight bash %}
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew update
$ brew install postgres
{% endhighlight %}
#####[source](http://exponential.io/blog/2015/02/21/install-postgresql-on-mac-os-x-via-brew/) 

##Tutorial
Once you have installed PostgreSQL on your machine, you should be able to access the psql interactive terminal by simply typing:
{% highlight bash %}
$ psql
{% endhighlight %}
And to exit the shell:
{% highlight bash %}
postgres=# \q
{% endhighlight %}
You are given a user by default, typically named postgres.  If exited, reenter the postgres interactive terminal:
{% highlight bash %}
$ psql
{% endhighlight %}
And once in the shell, list your postgres users:
{% highlight bash %}
postgres=# SELECT * FROM pg_user;
{% endhighlight %}
Press q to exit.

###Create new user with password authentication
From OS X terminal:
{% highlight bash %}
$ CREATEUSER -d -l -P -s yourNewUsername
{% endhighlight %}
And enter password.

__OR__:
{% highlight bash %}
$ CREATEUSER --interactive
{% endhighlight %}
And enter role (equivalent to yourNewUsername), superuser: y, and password.

Once you have created a new user with a password, lets create a database with the same name as your username (user/role) so we can login to it.  

From the Mac OS X terminal:
{% highlight bash %}
$ createdb sameAsUsernameJustMade
{% endhighlight %}
__Or__ from the psql interactive terminal:
{% highlight bash %}
postgres=# CREATE DATABASE sameAsUsernameJustMade;
postgres=# \q
{% endhighlight %}

Now, login as yourNewUsername from Mac OS X terminal:
{% highlight bash %}
$ psql -U yourNewUsername
{% endhighlight %}
And you should see yourNewUsername as the current user in place of what it was before (again, typically postgres or Home).
{% highlight bash %}
yourNewUsername=# SELECT * FROM pg_user;
{% endhighlight %}
And we should see both our default user and new user.  Press q to exit list. 

###Create database
Let's create a database in yourNewUsername psql shell:
{% highlight bash %}
$ psql -U yourNewUsername
yourNewUsername=# CREATE DATABASE sampleDB;
{% endhighlight %}
List all databases:
{% highlight bash %}
yourNewUsername=# \l
{% endhighlight %}
Under name should be sampleDB and the Owner should be yourNewUsername.  Similarly, the db you created with the same username as yourNewUsername should have the Owner of the default user provided upon installation.

To check what you're connected to at any given time:
{% highlight bash %}
yourNewUsername=# \c 
{% endhighlight %}
To connect to a different database:
{% highlight bash %}
yourNewUsername=# \c nameOfDatabase
{% endhighlight %}
###Drop database
From the Mac OS X terminal:
{% highlight bash %}
$ dropdb nameOfDatabase
{% endhighlight %}
__Or__ from the psql interactive terminal:
{% highlight bash %}
postgres=# DROP DATABASE nameOfDatabase;
{% endhighlight %}

###Create table
Let's connect to our sampleDB database and create two new tables; 'users' with usernames and passwords, and 'comments' with a user_id (reference to an id in 'users') and content.  Both will haveas an id that will be a serial primary key, meaning each id will be unique and non-recurring:
{% highlight bash %}
yourNewUserName=# \c sampleDB
sampleDB=# CREATE TABLE users (id serial primary key, username varchar(60), password varchar(60));
{% endhighlight %}
A success message will apper, CREATE TABLE, once entered.  Now for the 'comments' table:
{% highlight bash %}
sampleDB=# CREATE TABLE comments (id serial primary key, user_id integer references users(id) on delete cascade, content varchar(200));
{% endhighlight %}
Now to view our tables within the sampleDB:
{% highlight bash %}
sampleDB=# \dt
{% endhighlight %}
###Update table

####Add rows
To insert three rows into the 'users' table:
{% highlight bash %}
sampleDB=# INSERT INTO users VALUES ( default, 'a username', 'a password'), ( default, 'another username', 'another password'), ( default, 'a third username', 'a third password');
{% endhighlight %}
Where the order of values within the parentheses correspond to the columns of the given table.

To insert two rows into the 'comments' table:
{% highlight bash %}
sampleDB=# INSERT INTO comments VALUES ( default, (SELECT id FROM users WHERE username = 'a username'), 'some comment'), ( default, (SELECT id FROM users WHERE username = 'a third username'), 'some comment');
{% endhighlight %}

####Delete rows
To delete the row in 'users' where id = 2 (i.e. the second row):
{% highlight bash %}
sampleDB=# DELETE FROM users WHERE id = 2;
{% endhighlight %}
To delte the row in 'users' where username = 'a username':
{% highlight bash %}
sampleDB=# DELETE FROM users WHERE username = 'username';
{% endhighlight %}
To delete the row in 'comments' where user_id = 3:
{% highlight bash %}
sampleDB=# DELETE FROM comments WHERE user_id = 3;
{% endhighlight %}
###Dop table
For the 'users' table:
{% highlight bash %}
sampleDB=# DROP TABLE IF EXISTS users cascade;
{% endhighlight %}
For the 'comments' table:
{% highlight bash %}
sampleDB=# DROP TABLE IF EXISTS table cascade;
{% endhighlight %}
Cascade is only necessary if there are references from one table to another. If 'comments' didnt exist we could drop 'users' by simply:
{% highlight bash %}
sampleDB=# DROP TABLE IF EXISTS users;
{% endhighlight %}
