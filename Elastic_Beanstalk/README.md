# Set up Elastic Beanstalk Environment

Amazon has made deployment easy with their Elastic Beanstalk Command Line Interface (EBCLI). It’s available on PyPi, so in your virtual environment type:

```$ pip install awsebcli```

Once that finishes installing, we can initialize and deploy our Flask app. We could do this using our AWS root access. However, let’s follow [AWS Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) and create a new user for this demo. This will keep our master ID and secret key (which you should have stored somewhere) safe.