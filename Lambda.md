# Lambda

## Basics

### serverless compute

### pay per request

- per call

	- 1 million invocations free

		- 0.20$ / 1 Million requests 
		- 0.0000002$ / request

- per duration

	- 400,000 GBs free

		- 1.00$ / 600,000 GB sec

### integrated with AWS

- build in Cloudwatch

### all major languages

### upto 3GB RAM

- higher RAM -> higher CPU, N/W

### very cheap / very popular

### deploy code

- 50 MB compressed (max)
- 250 MB uncompressed (Max)

## Invocations

### Synchronous

- make a call

	- wait for response

- error handling at client
- types

	- Client / User invoked

		- CLI
		- SDK
		- API G/W
		- Application Load Balancer

			- as https endpoint
			- register as target group

				- via

					- API
					- CLI
					- Console

			- ALB -> LAMBDA

				- http -> JSON

					- n vice versa

			- Supports multi value headers

				- multiple query string

					- eg: http://<>/path?name=a&name=b

				- sent as JSON array

					- "queryStringParameters" : { "name" : ['a','b']}

			- Limits

				- Lambda / Target group in same region
				- req/response body max size 1 MB
				- websockets not supported

		- S3 batch
		- CloudFront (Lambda @Edge)

			- run Lambda alongside Cloudfront
			- deployed globally
			- modify

				- Viewer request

					- CloudFront

						- Origin request

							- Origin

				- Viewer response

					- CloudFront

						- Origin response

							- Origin

			- user case

				- eg. Dynamic webapp at edge
				- eg. Search Engine Optimization
				- eg. Bot mitigation
				- eg. A/B Testing
				- eg. Auth / Authr

	- Service

		- Cognito
		- Step Functions

	- Other

		- Lex
		- Alexa
		- Kinesis Data Firehose

### Asynchronous

- make a call

	- dont wait for response

		- from function code
		- Lambda still tells you if it received message 

	- placed in event queue

		- internal
		- 3 retries

			- wait 1 min/wait 2 min

	- Ensure processing is Idempotent

		- enforce in code

	- define DLQ

		- SNS or SQS

	- high processing

		- lambda handles scaling

- eg

	- S3

		- S3 Event Notification

			- SQS
			- SNS
			- Lambda

				- Invoke for specific actions

					- eg. ObjectCreate or ObjectDelete

			- can take upto a min
			- enable versioning

				- events possibly missed for simultaneous updates

	- SNS
	-  EventBridge

		- types

			- Pattern

				- link to AWS Service

					- eg. CodePipline

			- Cron

				- trigger event specifc time

		- Target Lambda
		- creates resource policy

			- permission service to invoke Lambda

	- CodeCommit
	- CodePipeline
	- CloudFormation
	- AWS Config
	- AWS IOT
	- CLI

		- --invocation-type Event

### Event Source Mapping

- Synchronous  Invocation
- Polled from source

	- batch

- types

	- Streams

		- types

			- Dynamo DB
			- Kinesis

		- iterator / shard

			- processed in order

				- trim horizon
				- latest
				- at timestamp

		- batch window

			- in seconds

		- split batch on error
		- multiple batches in parallel

			- 10 batches/shard
			- in-order processing / partition key

		- discarded events 

			- Destination

	- Queues

		- types

			- SQS
			- SQS FIFIO

		- Long Polling
		- batch size

			- messages / batch

				- 1-10 m/batches
				- 10 reccomended

		- set queue as trigger
		- scaling

			- SQS

				- 60 instances / min
				- 1000 batches

		- visibility timeout

			- reccomended -> 6 times processing time

		- discarded events

			- Dead letter queue

				- set up on SQS
				- (Lambda DLQ only for ASYNC)

			- Destinations

## Destinations

### Asynchronous

- Configure Success or Failure
- for

	- SQS
	- SNS
	- Lambda
	- Eventbridge 

### Event Source Mapping

- discarded event batches
- for

	- SQS
	- SNS

- can send directly to DLQ

	- Destinations preferred*

## Security

### Execution Role

- IAM Role
- Grants lambda permission to AWS resources
- sample managed policy

	- eg. AWSLambdaBasicExecutionRole
	- eg. AWSLambdaKinesisExecutionRole
	- ...

- Event Source mapping

### Resource Policy

- Other AWS service to invoke Lambda
- Similar to S3 bucket policy
- IAM princical

	- IAM Policy at principal
	- resource policy at Lambda

### cross account access

- use STS assumeRole API

	- target cross account ARN

- get credentials n target the resource

	- eg. s3

## Logging / Monitoring

### CloudWatch 

- logs

	- built in support
	- execution logs 
	- needs C/W execution role

- Metrics

	- invocations / durations / concurrent 
	- Error count / Success rate / Throttles
	- ASync delivery failures
	- Iterator age

		- Kinesis / DynamoDB

### X-Ray

- enable in Lambda config

	- Active Tracing

		- (Check box in CLI)

- triggers X-Ray Daemon

	- automatic

- X-Ray SDK in code
- X-Ray executionRole
- Env variables

	- _X_AMZN_TRACE_ID
	- AWS_XRAY_DAEMON_ADDRESS
	- AWS_XRAY_CONTEXT_MISSING

- role needs X-Ray permissions

## VPC

### AWS Owned

- Cannot access VPC

### assign

- VPCID
- subnets
- Security groups
- AWSLambdaVPCAccessExecutionRole
- Creates ENI in behind the scenes

	- IAM Role

		- needs EC2 ENI access

			- ec2:CreateNWINterface
			- ec2:DeleteNetworkInterface
			- ec2:DescribeNetworkInterface

### Internet Access

- Public subnet 

	- !=Internet Access
	- !=PublicIP

- Private Subnet

	- Assign NAT G/W
	- Internet G/W

### Public AWS Service

- Via VPC end point

## Configs

### RAM

- 128MB - 3GB
- bound to CPU
- 1792MB RAM ~ 1 CPU

	- >1792 MB = Use multi-threading in code

- if compute heavy, increase RAM

### Timeout

- default ~ 3 seconds
- Max -> 15 min

### Execution Context

- temp run time
- external dependencies initialized
- maintained in anticipation
- next run can reuse

	- initialize common code outside handler

- function handler

	- entry point
	- specify handler
	- declared as 

		- instance
		- static method

- includes /tmp

	- 512 MB of space
	- remains when context is frozen
	- not persistent

		- use S3

- env variables

	- 4KB Max

### Concurrency / Throttle

- - 1000 concurrent / account / region

	- can be increased

- Reserve concurrency

	- at function level

		- =limit

	- API

		- PutFunctionConcurrency
		- DeleteFunctionConcurrency

	- set to Zero to throttle

- invocations > limt

	- trigger throttle

		- sync inv -> 429 (429)
		- async inv -> retry automatically

			- if fails hit DLQ

	- applies to all functions in account

		- dev/prod alike

- Cold Start

	- Code loaded n initialised
	- dependecies take time to load 
	- first request takes longer
	- in VPC performance drastically improved

- Provisioned Concurrency

	- always running
	- no cold starts

		- serves with low latency

	- auto scaling to manage

		- schedule
		- target utilization

	- API

		- PutFunctionConcurrency
		- DeleteFunctionConcurrency

### Layers

- external depencies as layers
- reuse them in your code
- can share layers across lambda
- smaller app packages
- extracted /opt

### Versions / Alias

- Versions

	- editable version - $LATEST
	- publish when ready
	- immutable (Code + Config)
	- distinct ARN

- Alias

	- pointer to version

		- eg. dev / test / prod

	- mutable
	- enables stable configs

		- Event triggers
		- enables blue / Green deployments
		- destinations

	- Own ARNS
	- cannot refer other Aliases
	- used with CodeDeploy

		- automate alias traffic shift
		- integrated with SAM
		- deployment types

			- linear
			- canary
			- AllAtOnce

## Cloudformation

### inline functions

- Code: ZipFile

### S3 zipfile

- S3Bucket / S3Key / S3ObjectVersion

*XMind - Trial Version*