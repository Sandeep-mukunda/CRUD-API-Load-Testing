
## Lab Overview And High Level Design
Let's start with the High Level Design. 
![high-level-design](https://github.com/user-attachments/assets/0e85c9f7-6067-488d-af5b-8b3a455f3e60)
An Amazon API Gateway is a collection of resources and methods. For this tutorial, you create one resource (DynamoDBManager) and define one method (POST) on it. The method is backed by a Lambda function (LambdaFunctionOverHttps). That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

* Create, update, and delete an item.
* Read an item.
* Scan an item.
* Other operations (echo, ping), not related to DynamoDB, that you can use for testing.

The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation:

```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1",
            "name": "Bob"
        }
    }
}
```

The following is a sample request payload for a DynamoDB read item operation:

```json
{
    "operation": "read",
    "tableName": "lambda-apigateway",
    "payload": {
        "Key": {
            "id": "1"
        }
    }
}
```
## Setup
### Create Custom Policy
We need to create a custom policy for least privilege

1. Open policies page in the IAM console
2. Click "Create policy" on top right corner
3. In the policy editor, click JSON, and paste the following
<img width="1440" height="738" alt="Create-Policy" src="https://github.com/user-attachments/assets/a222e43d-c4cc-4417-b10d-aadd6e62f0e2" />

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

4. Give name "lambda-custom-policy", and click "Create policy" on botom right
### Create Lambda IAM Role
Create the execution role that gives your function permission to access AWS resources.

To create an execution role

1. Open the roles page in the IAM console.

2. Choose Create role.

3. Create a role with the following properties.

* Trusted entity type – AWS service, then select Lambda from Use case
* Permissions – In the Permissions policies page, in the search bar, type lambda-custom-policy. The newly created policy should show up. Select it, and click Next.
<img width="1429" height="737" alt="Create-Lamba-Role" src="https://github.com/user-attachments/assets/2697f546-5588-4668-bab2-f102f99849a5" />

* Role name – lambda-apigateway-role.
* Click "Create role"

### Create Lambda Function

To create the function

1. Click "Create function" in AWS Lambda Console

<img width="1438" height="229" alt="Create-Lambda" src="https://github.com/user-attachments/assets/b30a0782-feb8-41a5-a77b-e4597b4107b0" />


2. Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.13 as Runtime. Under Permissions, click the arrow beside "Change default execution role", then "use an existing role" and select lambda-apigateway-role that we created, from the drop down

3. Click "Create function"

<img width="1438" height="729" alt="Lambda-basic" src="https://github.com/user-attachments/assets/ed0667ff-cd66-4b8f-ad1c-1d088f4d9a3a" />

4. Replace the boilerplate coding with the following code snippet and click "Deploy"

**Example Python Code**

```python

from __future__ import print_function
import boto3
import json

print('Loading function')


def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
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
<img width="1438" height="734" alt="Lambda-code-paste" src="https://github.com/user-attachments/assets/efd50a87-eebd-47b5-b6c8-26657f767a3b" />

### Test Lambda Function

Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.

1. Click the "Test" tab right beside "Code" tab

2. Give "Event name" as echotest

3. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Save" to save

```json
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}
```
3. Click "Test", and it will execute the test event. You should see the output in the console

<img width="1439" height="735" alt="execute-test" src="https://github.com/user-attachments/assets/cc44f132-c3be-4ade-ab16-c889f8773c23" />
We're all set to create DynamoDB table and an API using our lambda as backend!

### Create DynamoDB Table

Create the DynamoDB table that the Lambda function uses.

**To create a DynamoDB table**

1. Open the DynamoDB console.
2. Choose "tables" from left pane, then click "Create table" on top right.
3. Create a table with the following settings.
Table name – lambda-apigateway
Partition key – id (string)
4. Choose "Create table".

<img width="1440" height="741" alt="create-dynam0-table" src="https://github.com/user-attachments/assets/027f01db-bd36-47d6-a137-b927bf6100a8" />

### Create API

**To create the API**

1. Go to API Gateway console
2. Click Create API
3. Scroll down and select "Build" for REST API
4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"

<img width="1436" height="741" alt="create-new-api" src="https://github.com/user-attachments/assets/573b453c-efbb-444c-bcd2-5656e3fe12ba" />

5. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next. Click "Create Resource"

6. Input "DynamoDBManager" in the Resource Name. Click "Create Resource"

<img width="1440" height="741" alt="create-resource-name" src="https://github.com/user-attachments/assets/4a5dc9ab-2d71-47c6-b317-7671c20b734f" />

7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, click "Create Method".

![Create-method](https://github.com/user-attachments/assets/0da60b3d-0be6-413b-837a-4d96e3e7e2c5)

8. Select "POST" from drop down.

9. Integration type should be pre-selected as "Lambda function". Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up.Select the function, scroll down and click "Create method".

![Create-lambda-integration](https://github.com/user-attachments/assets/e2903c77-2e2c-4fc8-ab31-6bf157420270)

Our API-Lambda integration is done!

### Deploy the API

In this step, you deploy the API that you created to a stage called prod.

1. Click "Deploy API" on top right

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Stage". Give "Prod" as "Stage name". Click "Deploy"

<img width="762" height="643" alt="deploy-api" src="https://github.com/user-attachments/assets/fa9afcc0-bf44-46de-9a61-92ef41948c14" />

3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint url. In the "Stages" screen, expand the stage "Prod", keep expanding till you see "POST", select "POST" method, and copy the "Invoke URL" from screen


![copy-invoke-url](https://github.com/user-attachments/assets/a5ea486d-66c3-4424-aed6-96a2e438bdaa)

### Running our solution

1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

```json
{
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}
```

2. To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.

* To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.

![create-from-postman](https://github.com/user-attachments/assets/da25c91b-875f-48e5-8cd8-b22cd6feceb2)

* To run this from terminal using Curl, run the below
```
$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
```

3. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Explore table items" button from top right, and the newly inserted item should be displayed.

<img width="1440" height="645" alt="dynamo-item" src="https://github.com/user-attachments/assets/c19918a0-be84-4aa6-ba1f-63cae52ccd27" />

<img width="1440" height="659" alt="dynamo-show-item" src="https://github.com/user-attachments/assets/19fd1a8e-04c4-4217-994b-e606c5bb5d21" />

4. To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table
```json
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```

<img width="899" height="435" alt="dynamo-item-list" src="https://github.com/user-attachments/assets/0e14214f-9ad0-4aa4-a5ff-04f259272ebf" />

We have successfully created a serverless API using API Gateway, Lambda, and DynamoDB!


### Cleanup
Let's clean up the resources we have created for this lab.

### Cleaning up DynamoDB
To delete the table, from DynamoDB console, select the table "lambda-apigateway", then on top right , click "Actions", then "Delete table"

To delete the Lambda, from the Lambda console, select lambda "LambdaFunctionOverHttps", click "Actions", then click Delete

To delete the API we created, in API gateway console, under APIs, select "DynamoDBOperations" API, click "Delete"





