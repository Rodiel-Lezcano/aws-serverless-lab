# 🚀 LevelUp! Serverless Lab: API Gateway + Lambda + DynamoDB

A hands-on lab to help you **build a serverless CRUD API** using AWS Lambda, API Gateway, and DynamoDB.

---

## 🧠 Lab Overview

This lab walks you through designing and building a simple REST-style API that supports DynamoDB operations (create, read, update, delete, list), backed by a Lambda function and exposed via Amazon API Gateway.

### Architecture Diagram

![High Level Design](./images/high-level-design.jpg)

### Supported Operations

The Lambda function handles the following operations based on your request payload:

- `create`, `read`, `update`, `delete` – DynamoDB item operations
- `list` – Scan table
- `echo`, `ping` – For basic testing/debugging

Example: **Create Item**
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

Example: **Read Item**
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

---

## 🛠️ Setup Instructions

<details>
<summary><strong>Step 1: Create IAM Policy</strong></summary>

1. Open the **IAM Console → Policies**.
2. Click **Create policy** → select the **JSON** tab.
3. Paste the following JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
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
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

4. Name it `lambda-custom-policy` and click **Create policy**.

![Create policy](./images/create-policy.jpg)

</details>

<details>
<summary><strong>Step 2: Create Lambda Execution Role</strong></summary>

1. Open **IAM Console → Roles** → **Create Role**
2. Trusted entity: **AWS Service** → Use case: **Lambda**
3. Attach policy: `lambda-custom-policy`
4. Role name: `lambda-apigateway-role`

![Create IAM Role](./images/create-lambda-role.jpg)

</details>

<details>
<summary><strong>Step 3: Create Lambda Function</strong></summary>

1. Open **Lambda Console → Create Function**
2. Author from scratch:  
   - Name: `LambdaFunctionOverHttps`  
   - Runtime: `Python 3.13`  
   - Existing role: `lambda-apigateway-role`

![Lambda basic info](./images/lambda-basic-info.jpg)

3. Replace the boilerplate code with the following:

```python
from __future__ import print_function
import boto3
import json

print('Loading function')

def lambda_handler(event, context):
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
        raise ValueError(f'Unrecognized operation "{operation}"')
```

![Lambda Code](./images/lambda-code-paste.jpg)

</details>

<details>
<summary><strong>Step 4: Test Lambda Function</strong></summary>

Use this event to test the function with a simple echo:

```json
{
  "operation": "echo",
  "payload": {
    "somekey1": "somevalue1",
    "somekey2": "somevalue2"
  }
}
```

![Execute Test](./images/execute-test.jpg)

</details>

<details>
<summary><strong>Step 5: Create DynamoDB Table</strong></summary>

- Name: `lambda-apigateway`
- Partition key: `id` (string)

![Create DynamoDB Table](./images/create-dynamo-table.jpg)

</details>

<details>
<summary><strong>Step 6: Create REST API in API Gateway</strong></summary>

1. Open **API Gateway Console → Create API → REST API → Build**
2. Name it `DynamoDBOperations`
3. Create Resource: `/DynamoDBManager`
4. Add `POST` method → Integrate with `LambdaFunctionOverHttps`

![Create API Resource](./images/create-resource-name.jpg)  
![Create Method](./images/create-method-1.jpg)  
![Integration](./images/create-lambda-integration.jpg)

</details>

<details>
<summary><strong>Step 7: Deploy the API</strong></summary>

1. Click **Actions → Deploy API**
2. Stage name: `prod`

![Deploy API](./images/deploy-api-2.jpg)  
![Copy Invoke URL](./images/copy-invoke-url.jpg)

</details>

---

## 🚀 Run & Validate

You can now invoke your API using:

### ▶️ Postman  
Set `POST` → Paste Invoke URL  
Body (raw, JSON):
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

![Postman Example](./images/create-from-postman.jpg)

### ▶️ Curl
```bash
curl -X POST -d '{"operation":"create","tableName":"lambda-apigateway","payload":{"Item":{"id":"1","name":"Bob"}}}' https://<API>.execute-api.<REGION>.amazonaws.com/prod/DynamoDBManager
```

### ✅ View Items in DynamoDB

- Go to the DynamoDB console → `lambda-apigateway` table → Explore items

![View Item](./images/dynamo-item.jpg)  
![Item Details](./images/dynamo-show-item.jpg)

---

## 🧹 Cleanup

To avoid ongoing charges, delete the resources:

- **DynamoDB Table:** `lambda-apigateway`
- **Lambda Function:** `LambdaFunctionOverHttps`
- **API Gateway API:** `DynamoDBOperations`

---

## 📚 Summary

In this lab, you:

- Created a secure, scalable serverless backend
- Integrated API Gateway + Lambda + DynamoDB
- Learned to invoke and test APIs using Postman/Curl
- Managed cleanup to avoid AWS charges

> 🎉 Congrats! You just built and tested a fully functional serverless stack.

---
