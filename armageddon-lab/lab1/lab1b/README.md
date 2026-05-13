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
- Incident response test

## Parameter Store

I added these values in Parameter Store:

stored values:
/lab/db/endpoint
/lab/db/port
/lab/db/name

These are just config values. They are not passwords.

The endpoint points to the RDS database: lab-mysql.csb6mem8ir7h.us-east-1.rds.amazonaws.com.
The port is: 3306.
The database name is: lab-mysql.

Secrets Manager

The database username and password were stored in Secrets Manager.

The secret name was:

lab/rds/mysql

This is where the app gets the database login from.

CloudWatch Logs

I changed the app service so the app writes logs to this file on EC2:

/var/log/rdsapp.log

Then I installed the CloudWatch Agent and sent that log file to CloudWatch Logs.

The CloudWatch log group was:

/aws/ec2/lab-rds-app

This helped because I could see app errors in AWS instead of only checking the EC2 instance.

Metric filter and alarm

I made a metric filter for:

"ERROR"

The metric was:

DBConnectionErrors

The namespace was:

Lab/RDSApp

Then I made a CloudWatch alarm named:

lab-db-connection-failure

The alarm triggers when there are 3 or more database connection errors in 5 minutes.

SNS

I created an SNS topic called:

lab-db-incidents

I added my email to it and confirmed the subscription.

This was used like a simple alert system.

Incident Response Section

For the incident response part, I simulated a credential problem.

I changed the password in Secrets Manager so it did not match the actual RDS password anymore.

After that, the app started failing when I went to:

/list

The browser showed a 500 error.

What I checked

I checked CloudWatch Logs and saw an error that said something like:

Access denied for user 'admin'

That told me the app could still reach the database, but the login was being rejected.

So this was not a network issue.
It was not a stopped database either.
It was a credential issue.

Failure type

The failure type was:

Credential failure / secret drift

The secret value and the real RDS password did not match.

Recovery

To recover, I corrected the password in Secrets Manager so it matched the real RDS password again.

I did not recreate EC2.
I did not recreate RDS.
I did not hardcode the password into the app.

After fixing the secret, I tested:

/list

and the notes showed again.

What this proved

This showed that the system could be recovered using the tools already in AWS.

The important part was using logs instead of guessing.

At first, all I had was a 500 error.
After checking the logs, I could see the real problem was the database password.

Evidence included

My evidence includes:

Parameter Store values
Secrets Manager metadata
CloudWatch log group
CloudWatch error logs
Metric filter
CloudWatch alarm
SNS topic/subscription
Recovery proof showing /list working again

Reflection questions

A) Why might Parameter Store still exist alongside Secrets Manager?
Parameter Store is good for normal config values, like endpoint, port, and database name. Secrets Manager is better for passwords.

B) What breaks first during secret rotation?
The app breaks first if the secret password and the real RDS password do not match.

C) Why should alarms be based on symptoms instead of causes?
Because at first I may not know the real cause. The symptom tells me something is wrong and I need to check it.

D) How does this lab reduce MTTR?
It makes recovery faster because I can use logs and saved values instead of guessing or rebuilding everything.

E) What would you automate next?
I would automate the CloudWatch Agent setup so EC2 starts logging right away when it launches.

SNS topic/subscription
Recovery proof showing /list working again
