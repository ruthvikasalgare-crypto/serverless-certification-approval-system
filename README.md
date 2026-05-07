# serverless-certification-approval-system
Event-driven Serverless Certification Approval System on AWS demonstrating Lambda, API Gateway, Step Functions, DynamoDB, SNS, and CloudWatch monitoring.

# Serverless Certification Approval System (AWS)

## Project Overview
Designed and implemented an end-to-end Serverless Certification Approval System using AWS services to automate approval workflows.

## Architecture
User Request → API Gateway → Lambda → Step Functions → SNS Notification → Approval Process

## AWS Services Used
- AWS Lambda
- API Gateway
- Step Functions
- DynamoDB
- SNS
- IAM
- CloudWatch

## Key Features
✅ Fully serverless architecture  
✅ Automated approval workflow  
✅ Event-driven processing  
✅ Secure IAM role-based access  
✅ Real-time notifications

## My Responsibilities
- Designed serverless architecture
- Developed Lambda functions (Python)
- Built workflow using Step Functions
- Implemented approval logic
- Monitoring using CloudWatch

## Skills Demonstrated
AWS | Serverless | Lambda | Step Functions | Automation | Cloud Architecture


Step 1: Create DynamoDB Table
Go to the DynamoDB console.
Click Create table.
Table name: CertificationRequests
Partition key: requestId (String)
Leave other settings as default and click Create table.
Wait for the table to become active.
Step 2: Create IAM Role for Lambda
Go to the IAM console.
Click Roles -> Create role.
Select AWS service and choose Lambda.
Click Next for permissions.
Add the following permissions policies (search and check):
AmazonDynamoDBFullAccess (For simplicity in this demo)
AWSStepFunctionsFullAccess (For simplicity)
CloudWatchLogsFullAccess
Click Next.
Role name: CertificationLambdaRole
Click Create role.
Step 3: Create Lambda Functions
You will create 4 functions. For each function:

Runtime: Python 3.4 (or latest)
Architecture: x86_64
Execution role: Use an existing role -> CertificationLambdaRole
3.1 Function: SubmitRequestFunction
Create function named SubmitRequestFunction.
Copy code from submit_request.py.
Configuration -> Environment variables:
Key: STATE_MACHINE_ARN
Value: (Leave blank for now, we will update this later)
3.2 Function: NotifyManagerFunction
Create function named NotifyManagerFunction.
Copy code from notify_manager.py.
3.3 Function: HandleApprovalFunction
Create function named HandleApprovalFunction.
Copy code from handle_approval.py.
3.4 Function: CheckStatusFunction
Create function named CheckStatusFunction.
Copy code from check_status.py.
Configuration -> Environment variables:
Key: TABLE_NAME
Value: CertificationRequests
Step 4: Create Step Functions State Machine
Go to the Step Functions console.
Click Create state machine.
Select Blank template or Write your workflow in code.
In the code editor, replace everything with the content of step-functions-json.
CRITICAL: You must update the placeholders in the JSON with your actual resource names/ARNs:
Replace ${DynamoDBTableName} with CertificationRequests (appears twice).
Replace ${NotifyManagerFunctionName} with NotifyManagerFunction (or the full ARN if cross-account, but function name works if in same account).
Click Next.
Name: ApprovalStateMachine
Permissions: Select Create a new role (Step Functions will automatically add permission to invoke Lambda and access DynamoDB based on your definition).
Click Create state machine.
Copy the ARN of the created State Machine (e.g., arn:aws:states:us-east-1:123456789012:stateMachine:ApprovalStateMachine).
Step 5: Update Lambda Configuration
Go back to Lambda console -> SubmitRequestFunction.
Configuration -> Environment variables.
Update STATE_MACHINE_ARN with the ARN you just copied.
Click Save.
Step 6: Create API Gateway
Go to API Gateway console.
Click Create API -> HTTP API (Build).
API name: CertificationAPI.
Integrations: You can add them now or later. Let's add them now.
Integration targets: Lambda
Choose SubmitRequestFunction, HandleApprovalFunction, CheckStatusFunction.
Configure routes:
POST /request -> SubmitRequestFunction
POST /approval -> HandleApprovalFunction
GET /request/{requestId} -> CheckStatusFunction
Click Next (Stages) -> Leave as $default.
Click Create.
Note the Invoke URL (e.g., https://xyz.execute-api.us-east-1.amazonaws.com).
Verification Steps
Submit a Request:

curl -X POST https://<YOUR-API-URL>/request \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "course": "AWS Certified Developer", "cost": 150}'
Response: {"requestId": "uuid...", "executionArn": "..."}

Check Logs for Token:

Go to CloudWatch Logs for NotifyManagerFunction.
Find the log entry APPROVAL TOKEN: .... Copy the long token string.
Check Status (Pending):

curl https://<YOUR-API-URL>/request/<REQUEST-ID>
Response: {"status": "PENDING", ...}

Approve Request:

curl -X POST https://<YOUR-API-URL>/approval \
  -H "Content-Type: application/json" \
  -d '{"requestId": "<REQUEST-ID>", "decision": "APPROVED", "taskToken": "<PASTE-TOKEN-HERE>"}'
Verify Status (Approved):

curl https://<YOUR-API-URL>/request/<REQUEST-ID>
Response: {"status": "APPROVED", ...}

## Author
Ruthvika Salgare
