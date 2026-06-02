# Amazon Bedrock MultiAgent

This repository contains an Amazon Bedrock multi-agent solution design with Lambda function code used as an **Amazon Bedrock Agent Action Group backend**.

The current implementation focuses on a leave balance retrieval use case where an Amazon Bedrock Agent invokes an AWS Lambda function to retrieve employee leave balance data from Amazon DynamoDB.

<img width="1408" height="874" alt="image" src="https://github.com/user-attachments/assets/c56b7da5-f27f-4d57-9dd5-d79479cccd7a" />

## Project Overview

This project demonstrates how Amazon Bedrock Agents can be connected to backend business data through AWS Lambda and Amazon DynamoDB.

The Lambda function acts as a deterministic backend tool. The Bedrock Agent handles the natural language interaction and invokes the Lambda function through an Action Group when leave balance information is required.

At a high level, the flow is:

```text
User request
    ↓
Amazon Bedrock Agent
    ↓
Action Group
    ↓
AWS Lambda function
    ↓
Amazon DynamoDB table
    ↓
Formatted Lambda response
    ↓
Amazon Bedrock Agent response
```

## Current Use Case

The current Lambda function supports an HR-style leave balance lookup.

A user may ask for leave or PTO balance information. The Amazon Bedrock Agent collects or passes an employee ID to the Lambda function. The Lambda then uses that employee ID to query DynamoDB and returns the leave balance data to the agent.

## Repository Contents

The current implementation includes Lambda function code for retrieving leave balance data.

Key file:

```text
lf_pto_balance
```

This file contains the Lambda handler logic for:

- Receiving the Bedrock Agent Action Group event
- Extracting the employee ID from the request body
- Querying DynamoDB for the employee record
- Formatting the response in the structure expected by Amazon Bedrock Agents
- Returning session attributes and prompt session attributes back to the agent

## Lambda Function Summary

The Lambda function performs the following steps:

1. Imports required libraries:

```python
import boto3
import json
```

2. Creates a DynamoDB client:

```python
client = boto3.client('dynamodb')
```

3. Receives the Amazon Bedrock Agent event through the Lambda handler:

```python
def lambda_handler(event, context):
```

4. Extracts the employee ID from the request body:

```python
user_input_data = event['requestBody']['content']['application/json']['properties'][0]['value']
```

5. Queries the DynamoDB table named `leaveBalanceHRTable`:

```python
response = client.get_item(
    TableName='leaveBalanceHRTable',
    Key={'empID': {'N': user_input_data}}
)
```

6. Retrieves the DynamoDB item:

```python
leave_balance = response['Item']
```

7. Formats the response body for the Bedrock Agent Action Group:

```python
response_body = {
    'application/json': {
        'body': json.dumps(leave_balance)
    }
}
```

8. Returns the final response using the structure expected by Amazon Bedrock Agents:

```python
api_response = {
    'messageVersion': '1.0',
    'response': action_response,
    'sessionAttributes': session_attributes,
    'promptSessionAttributes': prompt_session_attributes
}
```

## Architecture

```text
User
 ↓
Amazon Bedrock Agent
 ↓
Action Group
 ↓
AWS Lambda: lf_pto_balance
 ↓
Amazon DynamoDB: leaveBalanceHRTable
 ↓
Lambda response formatted for Bedrock Agent
 ↓
Amazon Bedrock Agent response to user
```

## AWS Services Used

This solution uses or is designed around the following AWS services:

- Amazon Bedrock Agents
- Amazon Bedrock Action Groups
- AWS Lambda
- Amazon DynamoDB
- AWS IAM
- Amazon CloudWatch Logs

## DynamoDB Table

The Lambda function expects a DynamoDB table named:

```text
leaveBalanceHRTable
```

The lookup key used by the Lambda function is:

```text
empID
```

The current code expects `empID` to be stored as a numeric DynamoDB attribute.

## Bedrock Agent Action Group Integration

The Lambda function is designed to work with an Amazon Bedrock Agent Action Group.

The expected input event includes fields such as:

- `requestBody`
- `actionGroup`
- `apiPath`
- `httpMethod`
- `sessionAttributes`
- `promptSessionAttributes`

The response returned by the Lambda function includes:

- `messageVersion`
- `response`
- `sessionAttributes`
- `promptSessionAttributes`

This structure allows the Lambda function to serve as a backend action that the Bedrock Agent can call during a conversation.

## Business Value

This project demonstrates a common enterprise AI pattern:

```text
Natural language request → AI agent → backend business system → grounded response
```

Potential business applications include:

- HR leave balance lookup
- Employee self-service assistants
- Warranty claim lookup
- Dealer support workflows
- Internal knowledge assistants
- Business process automation
- Enterprise data retrieval through natural language

## Security and Governance Considerations

For a production implementation, additional controls should be considered:

- Validate the employee ID before querying DynamoDB
- Add authentication and authorization controls
- Restrict Lambda IAM permissions to the required DynamoDB table only
- Avoid logging sensitive employee information
- Handle missing or invalid employee records
- Add input validation and error handling
- Monitor Lambda execution through CloudWatch
- Add least-privilege IAM policies
- Consider encryption and data privacy requirements

## Current Limitations

The current Lambda function is a learning or prototype implementation and has several improvement opportunities:

- Limited error handling
- Assumes the request body always contains the expected structure
- Assumes DynamoDB always returns an `Item`
- Does not handle employee-not-found scenarios
- Does not validate the employee ID
- Uses print statements for debugging
- The Lambda file does not currently use a `.py` extension
- No sample Bedrock Agent event is currently included
- No sample Lambda response is currently included
- No OpenAPI schema or Action Group schema is currently included

## Recommended Future Enhancements

Recommended next improvements:

- Rename `lf_pto_balance` to `lf_pto_balance.py`
- Add structured error handling
- Add sample Bedrock Agent event JSON
- Add sample Lambda response JSON
- Add Action Group OpenAPI schema
- Add IAM policy examples
- Add deployment instructions
- Add DynamoDB table setup instructions
- Add unit tests for Lambda event parsing
- Add CloudWatch logging guidance
- Organize the repo into folders such as:

```text
.
├── README.md
├── architecture/
│   └── bedrock_multi_agent_architecture.md
├── lambda/
│   └── lf_pto_balance.py
├── schemas/
│   └── action_group_schema.json
└── examples/
    ├── sample_bedrock_event.json
    └── sample_lambda_response.json
```

## Skills Demonstrated

This repository demonstrates practical exposure to:

- Amazon Bedrock Agent Action Groups
- AWS Lambda backend integration
- DynamoDB data retrieval
- Bedrock Agent request and response formatting
- Serverless application design
- Enterprise AI architecture
- Natural language access to business data
- Cloud-based agentic AI patterns

## Summary

This repository demonstrates how an Amazon Bedrock Agent can use an AWS Lambda Action Group to retrieve leave balance information from a DynamoDB table. The project represents an important enterprise AI architecture pattern: connecting natural language AI agents to backend systems through controlled, serverless tool execution.
