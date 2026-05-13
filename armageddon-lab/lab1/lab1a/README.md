# LAB1A - EC2 to RDS (AWS Console)

## What kind of lab is it?

In this lab I built a simple AWS setup where an EC2 instance runs a small Flask web app, and the app connects to an RDS MySQL database.

The main point of the lab was not really the app itself. The point was to understand how EC2 talks to RDS securely.

Basic configuration:

Browser -> EC2 Flask App -> RDS MySQL
Resources I used:

- EC2 instance: lab-ec2-app
- EC2 security group: ec2-lab-sg
- RDS database: lab-mysql
- RDS security group: rds-lab-sg
- Secret name: lab/rds/mysql
- IAM role: lab-ec2-secrets-role

## What I made

I launched an EC2 instance and used userdata to install and start a Flask app.

The app has these endpoints:


 `/init`
 `/add?note=example`
 `/list`

The RDS database was created with MySQL. The database is not publicly accessible.

The EC2 instance is public so I can reach the app in the browser.

Security group setup

The EC2 security group allows:

HTTP for `port 80` from `0.0.0.0/0`
SSH for `port 22` from my IP only

<img width="1563" height="457" alt="image" src="https://github.com/user-attachments/assets/00adbc27-7bc8-4524-9b73-75922b53502a" />


The RDS security group allows:

MySQL `port 3306` from the EC2 security group

<img width="1468" height="231" alt="image" src="https://github.com/user-attachments/assets/e2ac6a95-d021-4756-8f36-493c57297621" />


This was important because the database should not be open to the whole internet. Only the app server should be able to connect to it.

Secrets Manager

I stored the database credentials in AWS Secrets Manager.

Secret name:

lab/rds/mysql

The EC2 instance uses an IAM role to read the secret. The password is not hardcoded into the app.

Testing

I tested the app in the browser.

First I initialized the database:

http://3.90.40.21/init

Then I added notes:

http://3.90.40.21/add?note=first_note
http://3.90.40.21/add?note=hello
http://3.90.40.21/add?note=IAMpolicyConnectMe!!

Then I listed the notes:

http://3.90.40.21/list

<img width="1206" height="316" alt="Schermafbeelding 2026-05-11 193019" src="https://github.com/user-attachments/assets/e30f78a3-ccd8-41be-b932-d028cd0fea0f" />


The notes showed up which shoed the app could write to and read from the RDS.

Problems I ran into

At first the app was not reachable because the Flask service was not running correctly.

I checked the service with:

`sudo systemctl status rdsapp`

Then I checked the logs:

`sudo journalctl -u rdsapp --no-pager -n 100`

I found that boto3 was missing, so the app could not start. I simply deleted the current instance, and recreated another which properly applied the script and launched the app server.

I also had a database timeout issue. The RDS security group was allowing my own public IP instead of the EC2 security group. Once I changed the source to the EC2 security group, the app connected to RDS.

Evidence included

This folder includes screenshots and CLI evidence for:

- EC2 instance running
- EC2 IAM role attached
- RDS database available
- RDS security group allowing port 3306 from the EC2 security group
- Secrets Manager secret metadata
- /list output showing notes from the database

## What I learned

I learned how EC2 connects to RDS using security groups and IAM roles.

The biggest thing I learned is that a 500 error or timeout does not automatically mean the app is broken. It can also mean a security group, IAM role, or secret is wrong.


# Short answer

A Why is DB inbound source restricted to the EC2 security group?
- So only the EC2 server can reach the database. The database should not be open to outside traffic, because random people or malicious actors could try to connect to it.

B What port does MySQL use?
- MySQL uses port `3306`. In AWS it shows up as MySQL/Aurora.

C Why is Secrets Manager better than storing creds in code/user-data?
- Secrets Manager is better because the password is not sitting directly inside the app's code. If the password is hardcoded it will be is easier to leak by accident. With Secrets Manager the app can pull the secret when its needed.
