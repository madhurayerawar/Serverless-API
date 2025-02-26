# AWS Serverless-API
### Overview and Design

This repository demonstrates how to build a serverless API using AWS API Gateway, Lambda, and DynamoDB. It provides a simple RESTful API with single POST operation.

![image](https://github.com/user-attachments/assets/08f2cf4e-cf3e-4b4c-910e-2eef495e5c8a)

An Amazon gateway is an interface between clients and backend services, allowing users to expose AWS Lambda functions, HTTP endpoints, and other AWS services via RESTful and WebSocket APIs.
Here, we create one API named DynamoDBManager and one POST method on it backed by LambdaFunctionOverHttps a Lambda function. Lambda function is inserting the data in lambda-apigateway a DynamoDB database.

#### POST method DynamoDBManager supports below DynamoDB operations
- Create an item
- Read an item
- List all items
- Built-in logging and monitoring via CloudWatch

The JSON request which you send in the POST request identifies the operation and provides respective data. 

> ✅ **Tip:** Used Postman a thrid party application to send HTTP request to access DynamoDBManager API

  
## SETUP
### Create Lambda IAM Role
Configured an custom IAM role to give executional permissions to access AWS resources. To do so follow
1. Open IAM -> Policies in AWS console
2. Create a policy to give permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to DynamoDB and upload logs.
3. ## JSON

Below is a JSON for IAM policy:

```json
{
"Version": "2012-10-17",
"Statement": [
{
  "Sid": "Stmt1428341300017",
  "Action": [
    "dynamodb:DeleteItem",
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query",
    "dynamodb:Scan",
    "dynamodb:UpdateItem"
  ],
  "Effect": "Allow",
  "Resource": "*"
},
{
  "Sid": "",
  "Resource": "*",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Effect": "Allow"
}
]
}
```
4. Open IAM -> Roles to create a new role
     - Trust Entity - Lambda
     - Role name - **lambda-apigateway-role**
     - Attach above policy

### Create API Gateway


