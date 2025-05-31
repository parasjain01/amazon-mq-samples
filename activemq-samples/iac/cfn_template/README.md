# Amazon MQ CloudFormation Templates

This directory contains CloudFormation templates for Amazon MQ related resources.

## Templates

### Single Instance ActiveMQ Broker

The `single-instance-activemq-broker.yaml` template creates a single instance ActiveMQ broker with the following resources:
- VPC with public subnet
- Security group for the broker
- Single instance ActiveMQ broker
- Broker configuration
- SSM Parameter and Secrets Manager secret for broker credentials

### Bedrock Agent for MQ Migration Evaluation

The `bedrock-agent-mq-migration.yaml` template creates a Bedrock agent to evaluate migration of messaging services to Amazon MQ. The template includes:

- S3 bucket for agent artifacts and knowledge base documents
- IAM role for the Bedrock agent with appropriate permissions
- Knowledge base for storing information about messaging services and Amazon MQ
- Bedrock agent with instructions for evaluating migration
- Action groups for analyzing infrastructure, estimating costs, and planning migration

## Deployment Instructions

### Deploying the Bedrock Agent for MQ Migration Evaluation

1. **Prerequisites**:
   - AWS CLI installed and configured
   - Appropriate permissions to create CloudFormation stacks and Bedrock resources

2. **Deploy the CloudFormation stack**:
   ```bash
   aws cloudformation create-stack \
     --stack-name mq-migration-evaluation \
     --template-body file://bedrock-agent-mq-migration.yaml \
     --capabilities CAPABILITY_IAM \
     --parameters \
       ParameterKey=AgentName,ParameterValue=MQMigrationEvaluationAgent \
       ParameterKey=S3BucketName,ParameterValue=mq-migration-evaluation-agent-bucket-<unique-id>
   ```

   Replace `<unique-id>` with a unique identifier to ensure the S3 bucket name is globally unique.

3. **Upload knowledge base documents**:
   After the stack is created, upload relevant documents about messaging services and Amazon MQ to the S3 bucket in the knowledge base prefix:
   ```bash
   aws s3 cp ./knowledge-docs/ s3://mq-migration-evaluation-agent-bucket-<unique-id>/knowledge-base/ --recursive
   ```

4. **Sync the knowledge base**:
   ```bash
   aws bedrock-agent start-ingestion-job \
     --knowledge-base-id <knowledge-base-id> \
     --data-source-id <data-source-id>
   ```

   You can get the knowledge base ID from the CloudFormation stack outputs.

## Using the Bedrock Agent

Once deployed, you can interact with the Bedrock agent through:

1. **AWS Console**: Navigate to Amazon Bedrock > Agents > Your agent name
2. **API**: Use the Bedrock Runtime API to interact with the agent
3. **AWS SDK**: Use the AWS SDK to integrate the agent into your applications

The agent can help with:
- Analyzing your current messaging infrastructure
- Evaluating compatibility with Amazon MQ
- Estimating migration costs
- Creating a migration plan

## Implementation Notes

The action groups in the template reference Lambda functions that need to be implemented separately. These functions should:

1. **Analyze Infrastructure**: Evaluate the current messaging service and its compatibility with Amazon MQ
2. **Estimate Costs**: Calculate the costs of migrating to and running Amazon MQ
3. **Plan Migration**: Create a detailed migration plan with phases, tasks, and risk mitigation

You can implement these Lambda functions based on your specific requirements and integration needs.