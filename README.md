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

### Create API
Create DynamoDBOperations API following below steps
1. Go to API Gateway on AWS Console
2. Click Create API
   ![image](https://github.com/user-attachments/assets/374867bc-44fb-454c-841d-d3cc77fc1de7)
3. Scroll down and Select "Build" for Rest API
   ![image](https://github.com/user-attachments/assets/c0260951-a745-49f8-a951-6976e133e84b)
4. Give API name as "DynamoDBOperations" and click "Create API"
   ![image](https://github.com/user-attachments/assets/6dd781c0-74cd-42d6-817b-99f0ea9c8317)
   ![image](https://github.com/user-attachments/assets/542c8141-626f-497d-acbc-aee9b4beac85)

5. API is a collection of resources and methods are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Click "Create Resource"
   ![image](https://github.com/user-attachments/assets/011a0e3a-e6e8-4ea1-a11d-a29e993381e5)
6. Mention resource name "DynamoDBManager" and click "Create resource"
   ![image](https://github.com/user-attachments/assets/34f6e1f8-7528-4843-9310-c9f46a87a747)

7. Let's create a POST method for our API. Select "/DynamoDBManager" and click "Create Method"
   ![image](https://github.com/user-attachments/assets/553d5867-5016-49eb-af45-48cfcf196e25)
8. Select
   - Method type: POST
   - Integration type: Lambda
   - Lambda function: LambdaFunctionOverHttps (which we created earlier)
   - Click "Create method"
   ![image](https://github.com/user-attachments/assets/2b5b09ca-f203-45cd-ad5e-9faa7a5f9417)

  


### Create Lambda Function



