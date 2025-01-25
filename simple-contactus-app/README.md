# AWS Serverless Architecture: Step-by-Step Guide

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
3. Specify the table name (e.g., `ContactRequests`).
4. Set the **Primary Key** as `RequestId` (String).
5. Configure additional attributes as needed (e.g., `CustomerName`, `CustomerEmail`, `Message`).
6. Choose the default settings and click **Create Table**.

---

## Step 2: Create an AWS Lambda Function

1. Navigate to **AWS Lambda > Functions > Create Function**.
2. Choose **Author from scratch**.
   - Function Name: `ProcessContactRequest`
   - Runtime: Select `Node.js` or `Python`.
3. Under **Permissions**, attach the IAM role created earlier.
4. Add the following code:

### Example Code (Node.js):
```javascript
const AWS = require('aws-sdk');
const dynamoDb = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
    const data = JSON.parse(event.body);

    const params = {
        TableName: 'ContactRequests',
        Item: {
            RequestId: data.RequestId,
            CustomerName: data.CustomerName,
            CustomerEmail: data.CustomerEmail,
            Message: data.Message,
        },
    };

    try {
        await dynamoDb.put(params).promise();
        return {
            statusCode: 200,
            body: JSON.stringify({ message: 'Request processed successfully!' }),
        };
    } catch (error) {
        return {
            statusCode: 500,
            body: JSON.stringify({ message: 'Error processing request', error }),
        };
    }
};
```

5. Deploy the Lambda function.

---

## Step 3: Create an API Gateway

1. Navigate to **API Gateway > Create API**.
2. Choose **HTTP API**.
3. Define the API name (e.g., `ContactUsAPI`) and click **Create**.
4. Add a resource:
   - Resource Path: `/contact`
   - HTTP Method: `POST`
5. Configure integration:
   - Integration Type: **Lambda Function**
   - Lambda Function: Select `ProcessContactRequest`.
6. Deploy the API and note the endpoint URL.

---

## Step 4: Test the Application

1. Use **Postman** or `curl` to send a `POST` request to the API Gateway endpoint.
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

2. Check the DynamoDB table to verify that the data was inserted.

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
