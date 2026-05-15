# LAB1B - Secrets, and Incident Response

This lab builds on my Lab 1A setup.

In Lab 1A I had the EC2 app working with RDS.  
For Lab 1B I did not rebuild the app. I used the same EC2 instance and same RDS database, then added the operations parts around it.

The main idea of this lab was to see what happens after something is already deployed. I had to monitor it, break it, look at the logs, and recover it without recreating everything.

## What I added

For Lab 1B I added:

- Parameter Store values
- Secrets Manager check
- CloudWatch Logs
- CloudWatch Agent on EC2
- Metric filter
- CloudWatch Alarm
- SNS alert topic
- Incident response incident

## Parameter Store

I added these values in Parameter Store:

stored values:
/lab/db/endpoint
/lab/db/port
/lab/db/name

<img width="1865" height="601" alt="Schermafbeelding 2026-05-13 182646" src="https://github.com/user-attachments/assets/248249b6-1b34-4795-b55c-25dd8953b21e" />


These are just config values. They are not passwords.

The endpoint points to the RDS database: lab-mysql.csb6mem8ir7h.us-east-1.rds.amazonaws.com.
The port is: 3306.
The database name is: lab-mysql.

## Secrets Manager

The database username and password were stored in Secrets Manager.

The secret name was:

lab/rds/mysql

<img width="1806" height="821" alt="Secrets manager - lab-rds-mysql" src="https://github.com/user-attachments/assets/c12d7c99-fbb1-4e1b-8c60-21330a5081c9" />


This is where the app gets the database login from.

## CloudWatch Logs

I changed the app service so the app writes logs to this file on EC2:

/var/log/rdsapp.log

Then I installed the EC2 CloudWatch Agent and sent that log file to CloudWatch Logs.

<img width="784" height="321" alt="image" src="https://github.com/user-attachments/assets/246e981c-91bb-4da4-9292-82860b96cb67" />


The CloudWatch log group was:

/aws/ec2/lab-rds-app

This helped because I could see app errors in AWS instead of only checking the EC2 instance.

## Metric filter and alarm

I made a metric filter for:

"ERROR"

The metric was:

db-connection-error-filter
<img width="847" height="647" alt="metric-filter" src="https://github.com/user-attachments/assets/f545e644-9b2c-4455-990f-528bc9e42ff9" />


The namespace was:
## Lab/RDSApp


Then I made a CloudWatch alarm named:

lab-db-connection-failure

The alarm triggers when there are 3 or more database connection errors in 5 minutes.

<img width="1543" height="746" alt="Schermafbeelding 2026-05-13 215252" src="https://github.com/user-attachments/assets/235da11a-0202-415d-83ca-8d1a61baa139" />


## SNS

I created an SNS topic called:

lab-db-incidents

I added my email to it and confirmed the subscription.

This was used like an alert system.

<img width="1630" height="548" alt="sns-topic" src="https://github.com/user-attachments/assets/ebce43d1-698f-4802-89fd-ee30b6ae5644" />




Furthermore I changed the password in Secrets Manager so it did not match the actual RDS password anymore.

After that, the app started failing when I went to:

/list

The browser showed a 500 error.

<img width="1147" height="91" alt="image" src="https://github.com/user-attachments/assets/9e9bac9d-12be-41f3-8e87-58f0bb9073be" />


What I checked

I checked CloudWatch Logs and saw an error that said something like:

Access denied for user 'admin'

<img width="1895" height="859" alt="Schermafbeelding 2026-05-13 183014" src="https://github.com/user-attachments/assets/472328b3-b9b7-4900-8f0f-f33d550476d8" />


That told me the app could still reach the database, but the login was being rejected.

So this was not a network issue.
It was not a stopped database either.
It was a credential issue.

Failure type

The failure type was:

Credential failure / secret drift

The secret value and the real RDS password did not match.

## Recovery

To recover, I corrected the password in Secrets Manager so it matched the real RDS password again.

I did not recreate EC2.
I did not recreate RDS.
I did not hardcode the password into the app.

After fixing the secret, I tested:

/list

and the notes showed again.

<img width="795" height="317" alt="recovered-db-list-output" src="https://github.com/user-attachments/assets/d09af693-407b-419f-8247-392500b741ec" />

This showed that the system could be recovered using the tools already in AWS.

The important part was using logs instead of guessing.

At first, all I had was a 500 error.
After checking the logs, I could see the real problem was the database password.


# Incident Response Report

## Incident Summary

In this incident a component of the Database had been broken deliberately

A RDS inbound rule that let the EC2 security group connect on MySQL port `3306`

After I did that, the app was still loading, but `/list` stopped working because the app could not reach the database.

<img width="1027" height="131" alt="image" src="https://github.com/user-attachments/assets/00c332f6-b84a-4b0e-826e-e8b7bd47dfbb" />


## What I saw

I checked CloudWatch Log and the CloudWatch alarm.

<img width="388" height="140" alt="image" src="https://github.com/user-attachments/assets/1a9e680d-469a-4d53-9dac-6a3d773149d2" />

<img width="1104" height="479" alt="image" src="https://github.com/user-attachments/assets/91a22185-229a-444f-816c-3ed72e2c0a21" />

The logs showed database connection errors. It looked like a timeout issue, so that made me think it was a security group issue.

<img width="1621" height="594" alt="Schermafbeelding 2026-05-15 131536" src="https://github.com/user-attachments/assets/ed7470ba-0dc6-4354-917c-0849ca8e000e" />


I did not think it was a password problem because the error was not `Access denied`. So, I checked if the security group rules were correct.


## Root Cause

The RDS security group was missing the rule that allowed the RDS traffic from the EC2 security group.

<img width="819" height="451" alt="image" src="https://github.com/user-attachments/assets/b94b85db-7cb1-47d2-b871-b2e4ab686ca2" />


So the EC2 was blocked from communication to the RDS.

## Recovery

I added the RDS inbound rule back:

MYSQL/Aurora `TCP 3306` from the EC2 security group

Then I tested `curl http://32.198.12.95/list` again and the notes came back.

<img width="1023" height="61" alt="image" src="https://github.com/user-attachments/assets/44a5e538-deba-41aa-be50-ff15685d2d6c" />



It took me about an 1 hour an 30 minutes to resolve the problem

Evidence included

My evidence includes:

- Parameter Store values
- Secrets Manager metadata
- CloudWatch log group
- CloudWatch error logs
- Metric filter
- CloudWatch alarm
- SNS topic/subscription
- Recovery proof showing /list working again


## Reflection questions

A Why might Parameter Store still exist alongside Secrets Manager?
- Parameter Store is good for normal config values, like the endpoint, port, and database name. Secrets Manager is better for passwords.

B What breaks first during secret rotation?
- The app breaks first if the secret password and the real RDS password do not match.

C Why should alarms be based on symptoms instead of causes?
- Because at first I may not know the real cause. The symptom tells me something is wrong and I need to check it.

D How does this lab reduce MTTR?
- It makes recovery faster because I can use logs and saved values instead of guessing or rebuilding everything.

E What would you automate next?
- I would automate the CloudWatch Agent setup so EC2 starts logging right away when it launches.

Preventive Action - Incident Response

I would be more careful with database security group changes, and also add some kind alert when a database security group rule is changed.

