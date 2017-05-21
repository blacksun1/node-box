# node-box

Create a box for hosting a node application using cloud formation - or take the
script and use it anywhere else.

## Usage

### Create the infrastructure

My favourite way of using it is just using the command line... something like
this

```bash
aws cloudformation create-stack \
	--template-body file:////git//node-box//cf//create_server.yaml \
	--stack-name nodebox \
	--parameters \
		ParameterKey=MongoDbAdminUser,ParameterValue=mongo \
		ParameterKey=MongoDbAdminPassword,ParameterValue=MySuperSecretPassword123 \
		ParameterKey=KeyName,ParameterValue=myPrivateKey && \
aws cloudformation wait stack-create-complete --stack-name nodeUp && \
aws cloudformation describe-stacks --stack-name nodeUp && aws cloudformation describe-stack-events --stack-name nodeUp --query 'StackEvents[].{ResourceStatus:ResourceStatus,ResourceType:ResourceType,ResourceStatusReason:ResourceStatusReason}' --output table
```

but use it how you want really

### Testing that it worked.

Once you have the public URL of the ec2 instance which is available from the
stack output you should be able to ssh into the server using

```bash
ssh -i ~/.ssh/yourPrivateKey ubuntu@<ec2_dns_name>
```

You should also be able to access the following url's:

* You should get a 404: `http://<ec2_dns_name>/i_dont_exist.html`
* A 200. A single image: `http://<ec2_dns_name>/image.jpeg`
* A 200. A plain text message coming from a simple node app: `http://<ec2_dns_name>/app/helloWorld`

If it didn't then check the file `/var/log/cloud-init-output.log` to see what
went wrong.

### Put your content on the box

This script implements all of the hard infrastructure stuff but of course can't
cater to every application.

Here is where you should put your own stuff:

#### Mongo

A single Admin user has been added to the machine with a username and password
that you provide in the `MongoDbAdminUser` and `MongoDbAdminPassword`
parameters. To login then use the following

```bash
mongo -u mongo -p --authenticationDatabase admin
```

Create yourself a database and with a user account and put the details into
your applications configuration.

#### Web content

I'm guesing you have a web application that is using React/Angular/Vue or
whatever else is cool right now.

It goes into `/usr/node/www` like so

```bash
sudo su node
cd ~/www/
# Put your content here!
```

#### API

Your api is expected to be written in NodeJS and can go into the API folder.
Your application will need to be setup with PM2 before it will work though

```bash
sudo su node
cd ~/api/
# Put your application in here

pm2 delete testme
pm2 start YourApplicationGoesHere
pm2 save # Make sure that PM2 knows about your application after a reboot
```

