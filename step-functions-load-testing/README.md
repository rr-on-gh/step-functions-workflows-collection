# Serverless load generator using distributed map 

This project creates a serverless load generator Step Function and uses distributed map to load test a workload. This pattern load tests a Lambda function, however this can be extended to load test other workloads like HTTP endpoints, APIs or other services that Step Functions support.


It creates two AWS Step Functions. The first, `LoadTesterStateMachine`, orchestrates the ramp up of the load, checking the concurrency and holding the load for a specified period of time. The second step function, `LoadTesterStateMachineRunLoad`, is responsible for generating the load by using a distributed map and invoking the workload that needs to be load tested.

The step function `LoadTesterStateMachine` is the entry point and takes in 3 parameters:

 - `rampUpDuration` The time in minutes over which the load should gradually be increased till it reaches the `targetConcurrency`
 - `duration` The time in minutes after the `rampupDuration` the load test should run
 - `targetConcurrency` The total number of simulated concurrent virtual users that invoke the workload in parallel


Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Requirements

* [Create AWS Account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) in case you do not have it yet or log in to an existing one
* An IAM user or a Role with sufficient permissions to deploy and manage AWS resources
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) (AWS SAM) installed
* A Lambda function that you need to load test. This could be an existing Lambda function, or you could [create a new Lambda function](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html#getting-started-create-function).

## Deployment Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
   ```
   git clone https://github.com/aws-samples/step-functions-workflows-collection
   ```
1. Change directory to the workflow directory:
   ```
   cd step-functions-load-testing
   ```
1. From the command line, use AWS SAM to build and deploy the AWS resources for the workflow as specified in the template.yaml file:
   ```
   sam build
   sam deploy --guided
   ```
   1. During the prompts:

      - Enter a stack name
      - Enter the desired AWS Region
      - For the Parameter `LambdaFunctionName`, enter the ARN of the Lambda function that you want to load test
      - For the Parameter `LambdaParameters`, enter the parameters that need to be passed to the Lambda function
        - For example, if your lambda function accepts this input
        ```json
         {
           "s3Bucket": "my-s3-bucket",
           "s3Key": "photo.jpeg"
         }
         ```
        escape it appropriately:
        ```shell
           Parameter LambdaParameters []: {  \"s3Bucket\": \"my-s3-bucket\",  \"s3Key\": \"photo.jpeg\" }
        ```
      - Allow SAM CLI to create IAM roles with the required permissions.

      Once you have run `sam deploy --guided` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy` in future to use these defaults.

1. Note the outputs from the SAM deployment process. These contain the resource names and/or ARNs which are used for testing.

## How it works

![image](./resources/stepfunctions_graph.png)

## Testing
1. Navigate to the Step Function page on the AWS console and locate the `LoadTesterStateMachine` step function
2. Click on the "Start execution" button
3. On the pop-up window, use the below JSON as the input and hit "Start execution" button. This will start a load test that will simulate 5 concurrent users, ramped up over 1 minute and will hold the load for 5 minutes. 

```json
   {
      "rampUpDuration": 1,
      "duration": 2,
      "targetConcurrency": 5
   }
```
4. Navigate to the Lambda function that you are load testing and check the "Monitor" tab to view how the Lambda behaves under load

## Cleanup
 
1. Navigate to the root of the repository.
1. Delete the stack
    ```
    sam delete
    ```

----
Copyright 2024 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0
