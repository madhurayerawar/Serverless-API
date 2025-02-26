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
3. Below is a JSON for IAM policy:
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


### Create Lambda Function
1. GO to Lambda on AWS Console and click "Create API"
   ![image](https://github.com/user-attachments/assets/3cd2f1c2-4719-45e5-b457-c3466cfc0814)
2. Select "Author from Scratch"
3. Give Function name as "LambdaFunctionOverHttps"
4. Select Runtime language as "Python 3.13"
5. Architecture as x86_64
6. Click "Create function"
  ![image](https://github.com/user-attachments/assets/f784a54b-5f06-4c01-b1df-b33a99b7e3cd)

7. Click on the Lambda function and replace the boilerplate coding with below code
   ```
   from __future__ import print_function

   import boto3
   import json

     print('Loading function')
    def lambda_handler(event, context):
    #Provide an event that contains the following keys:

    #  - operation: one of the operations in the operations dict below
    #  - tableName: required for operations that interact with DynamoDB
    #  - payload: a parameter to pass to the operation being performed
    
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
   ```

### Test Lambda Function
Let's test our Lambda function before delploying and as we haven't created our API and DynamoDB so this will be just echo test. Hence, function should output whatever input we pass
1. Click on the "Test" tab on the Lambda function
2. Paste the below JSON
```
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```
3. Click "Save"
![image](https://github.com/user-attachments/assets/915f7a6c-8d98-4b4b-b709-fd7e4fe84e1c)

4. Click "Test" in the "Code" tab
![image](https://github.com/user-attachments/assets/cc841ea5-1fa4-4a92-b3ea-db22374a6434)

5. Once you are happy with the testing then Click "Deploy"
![image](https://github.com/user-attachments/assets/af21904f-9c5a-450f-87bf-80368d2a8e5d)

#### We are all set to create DynamoDB table and API using our Lambda as backend!


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


### Create Amazon DynamoDB Table 
Create the DynamoDB table that the Lambda function uses.
#### Follow below steps to create DynamoDB table
1. Go to DynamoDB service on AWS console
2. Select "Create table"
   ![image](https://github.com/user-attachments/assets/a0e1291c-23dc-475d-bdf3-76c5acad2389)
3. Provide
     Table name: lambda-apigateway
     Primary Key/ Partition Key: id (string)
4. Select "Create table"
   ![image](https://github.com/user-attachments/assets/8f2d0051-e116-4956-a09e-e20e369bc06a)
   ![image](https://github.com/user-attachments/assets/32358dbe-effc-498e-8210-2033ce5bd4e7)
![image](https://github.com/user-attachments/assets/462f5f2f-8bd4-4cf6-9c8e-90ce0669763f)


## Access and test API using Postman application
Postman is primarily used for testing and developing APIs, allowing developers to easily create, send, and analyze API requests, making it a key tool for managing the entire API lifecycle, from design and testing to documentation and collaboration. 
Follow below steps to access our API:
  1. Create a Balnk Collection which is nothing but like creating a folder
  2. In collection create new request by clicking ... and click "Add Request"
  3. Select method "POST" from drop down
  4. Add the API URL from the "Invoke URL" attribute of the API POST method in the PROD stage.
     ![image](https://github.com/user-attachments/assets/b2b049ff-9fc8-4c9a-b1f6-5dd8407c89be)
     ![image](https://github.com/user-attachments/assets/77e83ed4-f47f-4423-9f03-0e807ba042ac)
     ![image](https://github.com/user-attachments/assets/61dee301-a584-42ff-a1cc-36bddcb0b843)

     


     


