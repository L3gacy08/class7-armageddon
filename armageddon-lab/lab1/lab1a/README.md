# Lab 1 - EC2 to RDS

This folder contains my work for Lab 1A.

Lab 1A was focused on getting a basic EC2 web app connected to an RDS MySQL database.  

## Lab 1A - EC2 to RDS(MySQL Notes App

### What I made

For Lab 1A I created a prepared a Flask notes app running on an EC2 instance. The app connects to an RDS MySQL database and stores notes there.

The setup is:

Browser -> EC2 Flask App -> RDS MySQL

The EC2 instance is public so I can reach it in the browser.
The RDS database is not public. It only allows MySQL traffic from the EC2 security group.

Resources that were used:
EC2 instance: lab-ec2-app
EC2 security group: ec2-lab-sg
RDS database: lab-mysql
RDS security group: rds-lab-sg
Secret name: lab/rds/mysql
IAM role: lab-ec2-secrets-role

What worked:
I was able to open the app in the browser, initialize the database, add notes, and list notes from RDS.

The main working endpoints were:

/init
/add?note=first_note
/list
Lab 1A proof included in the lab1a deliverables

In the lab1a deliverables folder I included screenshots and CLI evidence for:

RDS security group allowing port 3306 from the EC2 security group
A IAM role attached to the EC2
RDS instance available and not public
Secrets Manager secret metadata
/list output showing notes from the database
And some issues

At first, the app was not reachable because the Flask service was not running correctly. Then I found that boto3 was missing, so the app could not start. I had decided to delete the instance and create new resource because it was simply due to a miss timed launch from the EC2 instance, and afterwards worked without an issue.

I also had a database connection timeout because the RDS security group was allowing my home IP instead of the EC2 security group. Once I changed the source to the EC2 security group, /init worked.
