# IDS721-Proj4
This is my study for AWS DynamoDB, SQS, S3, AWS Lambda, AWS Comprehend to build Data Engineering Pipeline

## Overview WorkFlow

<img width="1263" alt="serverless" src="https://user-images.githubusercontent.com/123284219/227793727-9118dc78-7b09-41f0-9de7-6f91ee029223.png">

## Set Up

To build this serverless architecture on AWS from scratch, follow these steps:

**Step 1: Create DynamoDB table, SQS queue, S3 bucket & working environment**

* Open AWS DynamoDB console and create a table called "fang". For the primary key, use "name" with type "string".

* Click on the newly created table and add items with names like "Apple", "Google", etc.

* Open AWS SQS console and create a standard queue called "readDB" .

* Open AWS S3 console and create a bucket called "fangsentimentsforids". 

* Open AWS Cloud9 console, open the IDE of your working environment, create a new directory for this project, and cd into it.

**Step 2: Create producer-side Lambda function (triggered by a CloudWatch timer)**

This function reads data from the DynamoDB table and puts messages into the SQS queue.


2.1. Create a SAM application

* In the Cloud9 IDE, click on "AWS", then right-click on "Lambda", and choose to "Create Lambda SAM Application".

* For settings of the SAM app, select Python 3.7/3.8 as the runtime, select "AWS SAM Hello World" as the SAM application template, choose the directory to store the files, and name your app "readDBValue".

* In the Cloud9 IDE, click on "Environment" and check the file hierarchy. Find your Lambda function's folder and open the sub-folder "hello-world".

* Replace "app.py" with the file lambda-producer/hello_world/app.py from this repository.

* Modify app.py, change the name of the DynamoDB table and the name of the SQS queue.

* Replace "requirements.txt" with the file lambda-serverless/hello_world/requirements.txt from this repository.

2.2. Build and deploy

Build the application using SAM:

```css
sam build --use-container
```

``` css
sam deploy --guided
```

**Step 3: Add Permission**

* Access the AWS Lambda console and locate the new app "sam-app" and a new Lambda function named "sam-app-HelloWorldFunction-xxxx". Click on the function, then navigate to "Configuration" > "Permission", and click on the existing role to be directed to the IAM console.

* In the new window, click "Attach Policies". On the subsequent page, find the "AdministratorAccess" policy and attach it.

* Return to the previous Lambda function page in the AWS Lambda console and select "Triggers". Remove the "API Gateway" trigger.

* Add a new EventBridge (CloudWatch Events) trigger. For its configuration, create a new rule and name it "OneMinuteTimer" (or any desired name). For the "schedule expression", input "rate(1 minute)".

**Step 4: Test lambda-producer**

* You can now enable the trigger and view the messages in the SQS queue.

* In the AWS SQS console, click on the "readDB" queue, followed by "Send and receive messages", and then "poll for messages".

* In the AWS Lambda console, on the specific function page, click on "Monitor" to find more details about the activity. You can also select "View Logs in CloudWatch".

You have the option to disable the trigger and purge the SQS queue at any time.


**Step 5: Create a second SAM application named "lambda-consumer" like the previous one**

* Replace "app.py" with the file lambda-consumer/hello_world/app.py from this repo.

* Modify app.py to update the REGION and bucket name.  
**ATTENTION: you need select REGION as us-east-1, otherwise, the AWS Comprehend will not be supported in other area like us-west-1**

* Replace "requirements.txt" with the file lambda-checkSQS/hello_world/requirements.txt from this repo.

**Step 6: Build and Deploy**

* Run sam build --use-container.
* Run sam deploy --guided.

**Step 7: Add Permission & Trigger to lambda-consumer function**

* Add permission and remove the "API Gateway" trigger as done previously.

* Add a new SQS trigger with your queue selected and a batch_size of 1.

* Modify Lambda Function Timeout. 

To successfully write results to S3, modify the timeout for the second Lambda function:

* Open the second Lambda function's page, click on "Configuration", and then click on "General configuration".
* Set the timeout to 1 minute.

**Step 8: Test Overall Pipeline**

You can now activate both triggers for the two functions and view the output in the S3 bucket, where you will find a csv file with the sentiment analysis output

Clear the activity by disabling the triggers and purging the SQS queue

<img width="1256" alt="projresult" src="https://user-images.githubusercontent.com/71317967/231502667-df3bd48b-b84f-4f75-809f-2496606e34e8.png">


## Demo Video

https://user-images.githubusercontent.com/71317967/231378199-6c4c1be3-8a96-494b-b0b0-0f83d032d53d.mp4

## References
[Source code](https://github.com/noahgift/awslambda)
