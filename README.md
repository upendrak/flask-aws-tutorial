# Deploying a Flask application in AWS: An end-to-end tutorial

This is the code that goes along with the detailed writeup here https://medium.com/@rodkey/deploying-a-flask-application-on-aws-a72daba6bb80

It's a simple Flask app that writes and reads from a database. It uses Amazon RDS for the database backend, but you can make things even simpler and use a local DB. For this tutorial, I am going to use Amazon RDS.

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

An example is shown in `config.py` file in the repo

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

# elasticbeanstalk-mysql-rds-flask app

## Set up Elastic Beanstalk Environment

Amazon has made deployment easy with their Elastic Beanstalk Command Line Interface (EBCLI). It’s available on PyPi, so in your virtual environment type:

```
$ pip install awsebcli
```

Once that finishes installing, we can initialize and deploy our Flask app. We could do this using our AWS root access. However, let’s follow [AWS Best Practices](http://docs.aws.amazon.com/IAM/latest/UserGuide/IAMBestPractices.html) and create a new user for this demo. This will keep our master ID and secret key (which you should have stored somewhere) safe.

To create a new user, go to the AWS Console and search for Identity and Access Management (IAM) and select “Create New Users”

We’ll only need one user for this demo, and we’ll call him/her “flaskdemo”:

Select “programmatic access” click “Next” for permissions.

We’ve created a user, but they have no permissions (just like our database). Let’s grant this user admin access. On this screen, select “Create group:”

In the next screen, search for “administrator” from the preconfigured options:

Many of these policies are specialized for specific AWS services. For simplicity choose the first one, “AdministratorAccess” and click “Create group”.

Now we can create our admin user

But we’re not done yet. The final step (and one that I sometimes forget) is to assign this group to our “flask” user. So back in the IAM main page, select “Users”, and then click “Add Permissions”:

We see our “admin” group:

Click “Next” and we’ve created an admin.
Now let’s finally initialize our Elastic Beanstalk environment. In your command window, type:

```
$ eb init

Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) cn-northwest-1 : China (Ningxia)
14) us-east-2 : US East (Ohio)
15) ca-central-1 : Canada (Central)
16) eu-west-2 : EU (London)
17) eu-west-3 : EU (Paris)
18) eu-north-1 : EU (Stockholm)
19) ap-east-1 : Asia Pacific (Hong Kong)
20) me-south-1 : Middle East (Bahrain)
(default is 3): 
```
Chose the location closest to you (I left mine as default). 

Next you’ll be prompted for the AWS ID and Secret Key for the user “flaskdemo” you saved somewhere:

You have not yet set up your credentials or your credentials are incorrect

You must provide your credentials.
(aws-access-id): <enter the 20 digit AWS ID>
(aws-secret-key): <enter the 40 digit AWS secret key>

Next you’ll see:

```
Enter Application Name
(default is "flask-aws-tutorial"): 
Application flask-aws-tutorial has been created.
```

Now the EBCLI just wants to make sure we’re using Python:

```
It appears you are using Python. Is this correct?
(Y/n): Y

Select a platform version.
1) Python 3.6
2) Python 3.4
3) Python 3.4 (Preconfigured - Docker)
4) Python 2.7
5) Python
(default is 1): 
```

You can leave it as default for this option

Next is Codecommit. You can leave this as default

```
Do you wish to continue with CodeCommit? (y/N) (default is n):
```

You have the option of creating an SSH connection to this instance. We won’t need to use it, so I recommend “no.” (If you need to ssh into this instance later, you can change the preferences of your EC2 instance from the AWS console later.)

```
Do you want to set up SSH for your instances?
(Y/n): n
```

Okay, now we’re all set up. Time to deploy this bad boy.

## Deploy our Flask Application

Before you deploy your flask application, make sure that you commit and push all your changes to your master branch on your remote (github).

Now your code is staged and ready for deployment. (And if you’ve modified any other files, do a “git add” on them too.). Before we create, we have to create an environment name and DNS CNAME for our app. The domain name is the most important — it will show up in the URL as http://<dns cname>.elasticbeanstalk.com. It has to be unique, or the command line will make your choose another.

From the command line, type:

```
$ eb create
Enter Environment Name
(default is flask-aws-tutorial-dev): 
Enter DNS CNAME prefix
(default is flask-aws-tutorial-dev22): flaskawsdemoapp

Select a load balancer type
1) classic
2) application
3) network
(default is 2): 
```

Once you’ve selected a unique DNS CNAME, you’ll see status updates as the app is deployed. They’ll look like this:

When the uploading finishes, you’ll see:

**2019-10-01 20:11:17    INFO    Application available at flaskawsdemoapp.us-west-2.elasticbeanstalk.com**

**2019-10-01 20:11:17    INFO    Successfully launched environment: flask-aws-tutorial-dev**

And that’s it. Hopefully this helps you get off and running with AWS. Finally, making updates to your site is easy. Whenever you update a file, simply type

```
$ eb deploy
```

when your new changes are ready. You can add images, email support, register your site with a .com (I recommend AWS Route 53 if you’re going to stay within AWS), and so much more.

Congrats on your first AWS site, and good luck!
