### Deploying a Flask application in AWS: An end-to-end tutorial

This is the code that goes along with the detailed writeup here:

https://medium.com/@rodkey/deploying-a-flask-application-on-aws-a72daba6bb80

It's a simple Flask app that writes and reads from a database. It uses Amazon RDS for the database backend, but you can make things even simpler and use a local DB.

To tool around with the app directly, here's a quickstart guide. 

Clone this repo to your local machine. In the top level directory, create a virtual environment:
```
$ pip install virtualenv
$ virtualenv flask-aws
$ source flask-aws/bin/activate
```
Now install the required modules:
```
$ pip install -r requirements.txt
```
To play with the app right away, you can use a local database. Edit ```config.py``` by commenting out the AWS URL and uncomment this line:
```
SQLALCHEMY_DATABASE_URI = 'sqlite:///test.db'
```

Next create a database:
```
$ mysql -u flask -h flaskinsight.crrvqov9dgv2.us-east-2.rds.amazonaws.com -p
Enter password: 
mysql> CREATE DATABASE BucketList;
```
> Replace the username and hostname (end point) from your own RDS configurations

Edit the `config.py` file to include the username, password, and db name you entered earlier, in the format:

```SQLALCHEMY_DATABASE_URI = ‘mysql+pymysql://<db_user>:<db_password>@<endpoint>/<db_url>’```

An example is show in `config.py` file in the repo

(Ignore this if you’re using a local database.)
  
Now create the tables in your (currently) empty database by running
```
$ python db_create.py
```

And the tables are created.  Now you can launch the app:
```
$ python application.py
```

And point your browser to http://0.0.0.0:5000 or http://localhost:5000

Using the top form, you can write to the database:

![Site main page](http://i.imgur.com/2d66GIB.png)

![Data entered](http://i.imgur.com/AQWdD2Q.png)

Get confirmation:

![confirmaton](http://i.imgur.com/JtemL7a.png)

Using the bottom form, you can see the last 1 to 9 entires of the database in reverse chronological order:

![results](http://i.imgur.com/LFJeKDz.png)
