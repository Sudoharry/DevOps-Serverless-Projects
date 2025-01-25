# AWS Serverless Architecture: Step-by-Step Guide
![main](https://github.com/user-attachments/assets/e1fbc6e1-41fd-49fa-89d9-cbb4b580280a)


This guide outlines how to build a serverless web application using AWS services:

- **API Gateway**: Serves as the front-end interface to handle HTTP requests.
- **Lambda**: Acts as a web server to process requests.
- **DynamoDB**: Functions as the database for reading and storing data.

## Process Flow
1. The customer accesses the web application.
2. They submit a request (e.g., filling out a Contact Us form).
3. The API Gateway receives the request and routes it to the appropriate API endpoint.
4. AWS Lambda processes the request and interacts with DynamoDB to store or retrieve data.

---

## Prerequisites

1. **AWS Account**: Ensure you have access to the AWS Management Console.
2. **IAM Role**: Create a role with permissions for Lambda to interact with DynamoDB and API Gateway.
3. **AWS CLI**: Install and configure the AWS Command Line Interface.
4. **Node.js or Python**: For writing the Lambda function code.

---

## Step 1: Create a DynamoDB Table

1. Go to the AWS Management Console.
2. Navigate to **DynamoDB > Tables > Create Table**.
3. Specify the table name (e.g., `harrytables`).
4. Set the **Primary Key** as `email` (String).
5. Configure additional attributes as needed (e.g., `CustomerName`, `CustomerEmail`, `Message`).
6. Choose the default settings and click **Create Table**.
---
![Create-table](https://github.com/user-attachments/assets/22226216-73d2-4c0f-aa1e-1991812c7521)

### Create a role for Lambda function:
---
![role-with-permission](https://github.com/user-attachments/assets/c9561948-720d-487f-b2d1-e7892692b75b)

---

## Step 2: Create an AWS Lambda Function

1. Navigate to **AWS Lambda > Functions > Create Function**.
2. Choose **Author from scratch**.
   - Function Name: `ProcessContactRequest`
   - Runtime: Select `Node.js` or `Python`.
3. Under **Permissions**, attach the IAM role created earlier.
4. Add the following code:

### Example Code (Python3):
```python3
import json
import os
import boto3

def lambda_handler(event, context):
    try:
        mypage = page_router(event['httpMethod'], event['queryStringParameters'], event['body'])
        return mypage
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }

def page_router(httpmethod, querystring, formbody):
    if httpmethod == 'GET':
        try:
            with open('contactus.html', 'r') as htmlFile:
                htmlContent = htmlFile.read()
            return {
                'statusCode': 200,
                'headers': {"Content-Type": "text/html"},
                'body': htmlContent
            }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps({'error': str(e)})
            }

    elif httpmethod == 'POST':
        try:
            insert_record(formbody)
            with open('success.html', 'r') as htmlFile:
                htmlContent = htmlFile.read()
            return {
                'statusCode': 200,
                'headers': {"Content-Type": "text/html"},
                'body': htmlContent
            }
        except Exception as e:
            return {
                'statusCode': 500,
                'body': json.dumps({'error': str(e)})
            }

def insert_record(formbody):
    formbody = formbody.replace("=", "' : '")
    formbody = formbody.replace("&", "', '")
    formbody = "INSERT INTO avinashtable value {'" + formbody + "'}"

    client = boto3.client('dynamodb')
    response = client.execute_statement(Statement=formbody)
    # Assuming the execute_statement call returns successfully
    return response
```
---
![Lambda-functions](https://github.com/user-attachments/assets/272a3377-0519-4ea8-bb3b-d1fcbdadaf49)

---

5. Deploy the Lambda function.

---

## Step 3: Create an API Gateway

1. Navigate to **API Gateway > Create API**.
2. Choose **HTTP API**.
3. Define the API name (e.g., `ContactUsAPI`) and click **Create**.
4. Add a resource:
   - Resource Path: `/`
   - HTTP Method: `POST`
   - HTTP Method: `GET`
5. Configure integration:
   - Integration Type: **Lambda Function**
   - Lambda Function: Select `ProcessContactRequest`.
6. Deploy the API and note the endpoint URL.
---
![Created-POST-METHOD](https://github.com/user-attachments/assets/62580c7d-53c0-468d-ab61-eb1a8222969e)

---
---
![Create-GET-METHOD-LAMBDA](https://github.com/user-attachments/assets/d3112ade-0a61-41b8-a5d3-f10bd94a457e)

---

## Step 4: Test the Application

1. Use **Postman** Invoke URL from API, or `curl` to send a `POST` request to the API Gateway endpoint.
   - URL: `https://<your-api-endpoint>/contact`
   - Headers: `Content-Type: application/json`
   - Body:
     ```json
     {
       "RequestId": "12345",
       "CustomerName": "John Doe",
       "CustomerEmail": "john.doe@example.com",
       "Message": "I am interested in your services."
     }
     ```
     ---
     ![Invoke-URL](https://github.com/user-attachments/assets/08dbdbf5-6560-471a-945d-0627d03e79f3)

     ---
     ![Contact-us-filling-form](https://github.com/user-attachments/assets/5002cab9-1d92-4af8-b77c-3615f66ed361)

     ---
     ---
     ![Successfull-submission-of-data](https://github.com/user-attachments/assets/d918e795-fb42-4c51-8f96-40ed191ff775)

     ---

2. Check the DynamoDB table to verify that the data was inserted.
---
![Tables-are-savedin-DynamoDB](https://github.com/user-attachments/assets/a03b775a-6750-4e94-a222-6183f0c2dcd0)

---

## Step 5: Add Monitoring and Logs

1. **CloudWatch Logs**:
   - Enable logging for Lambda and API Gateway.
2. **Metrics**:
   - Use CloudWatch to monitor invocations, latency, and errors.

---

## Optional Enhancements

- **Authentication**: Use AWS Cognito to secure the API Gateway.
- **Validation**: Add request validation to the API Gateway.
- **Error Handling**: Improve error responses in the Lambda function.
- **Frontend**: Create a simple frontend using React or HTML/CSS.

---

## Conclusion

You have successfully created a serverless architecture using AWS services. This setup can be scaled and extended based on your application needs. For further customization, explore additional AWS services such as SNS for notifications or S3 for file storage.
