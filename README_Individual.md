# Individual Project 4: AWS Lambda Step Functions

## Overview
The objective of this individual project was to build upon previous AWS lambda functions created and using a step function to orchestrate the lambda functions. The step function will be used to call the lambda functions in a specific order and pass the output of one lambda function to the next lambda function. The step function will be triggered by an API Gateway endpoint.


## Using SAM CLI to build and deploy the application
For a full tutorial on how to build and deploy the application, please refer to the [README.md](https://github.com/AaryaDesai1/Rust_Microservice/blob/main/README.md). 

## Creating a Step Function
The main part of this project was to create a step function and this section will walk you through the steps to create a step function and the possible caveats you might encounter.

### Step 1: Create a State Machine
To create a state machine, you can navigate to the AWS Management Console and search for `Step Functions`. Click on the `Create state machine` button and you will be taken to a page where you can define your state machine.
<img width="1720" alt="image" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/8435b561-3f10-4d8c-9d5d-cf809a65f65a">

### Step 2: Define the State Machine
You can define the state machine using JSON or the visual workflow editor. For this project, I used the visual workflow editor to define the state machine. Personally, it's much easier to work with. However, if you use generative AI to help you with your assignment, you should be able to get a pretty good JSON to work with as well (which I also used)! The state machine I created is shown below:
<img width="992" alt="Screenshot 2024-04-22 at 10 59 01 AM" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/7c66c18d-537e-483f-8a73-d5b33faef3f5">

I started with the above, extremely simple step function. However, I wanted to make it more comprehensive and added a few more states to it. For the same, I added some JSON to the state machine definition. The JSON is shown below:
```json
{
  "StartAt": "InvokeLambda",
  "States": {
    "InvokeLambda": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:905418434745:function:Rust-Serverless-Microservice-PutFunction-UAK78Ex7vK1x:$LATEST",
      "Next": "HandleResponse",
      "Catch": [
        {
          "ErrorEquals": [
            "States.ALL"
          ],
          "Next": "HandleError"
        }
      ]
    },
    "HandleResponse": {
      "Type": "Pass",
      "ResultPath": "$.response",
      "End": true
    },
    "HandleError": {
      "Type": "Fail",
      "Cause": "An error occurred during the execution of the Lambda function",
      "Error": "LambdaFunctionError"
    }
  }
}
```

This resulted in the following state machine:
<img width="1715" alt="image" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/89239374-32d4-49e4-805c-3b62b21525db">

This has the following states:
1. `InvokeLambda`: This state invokes the lambda function created in the previous project. The lambda function is invoked with the input provided in the `Input` field of the state machine. The output of the lambda function is stored in the `response` field.
2. `HandleResponse`: This state is a pass state that stores the output of the lambda function in the `response` field.
3. `HandleError`: This state is a fail state that is triggered if there is an error in the lambda function.

### Step 3: Configure the State Machine
First, you want to navigate to the `Config` tab of the state machine. Here, you can rename your state machine and provide a description for it. You can also provide a role that has the necessary permissions to execute the lambda function. For this project, I chose to `Create a new role` and provided the necessary permissions to the role. 
<img width="1313" alt="Screenshot 2024-04-22 at 10 58 52 AM" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/3d314480-6f37-4ffe-b27e-5756bcfac069">


After defining the state machine, you can configure the state machine by providing a name and a role that has the necessary permissions to execute the lambda function (this can be done on the same page as the `Design` tab by clicking on the `Lambda:Invoke` component of the flow chart). For this, you may want to look at your lambda function through the console and double check the name of the role that is associated with the SAM lambda function created.
**Note**: Since I used a JSON to define the state machine, the function was automatically added to the state machine. However, if you use the visual workflow editor, you will have to add the lambda function to the state machine manually.
**AUTOMATIC ADDITION**
<img width="1418" alt="image" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/abe43f6f-b578-4c62-83c0-9cb9473bdf8f">

**MANUAL ADDITION**
<img width="1702" alt="image" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/6fe4d213-8ef9-488b-b062-740effa65a75">

After all this, you can click on the `Create state machine` button to create the state machine. You'll get this prompt, and you can go ahead and `Confirm`:
<img width="1594" alt="Screenshot 2024-04-22 at 10 59 10 AM" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/9031f1bf-4ccf-413f-bc52-8a1d3bd892f5">


### Step 4: Test the State Machine

To test the state machine, you can click on the `Start execution` button. You can provide an input to the state machine and start the execution. The input provided to the state machine is passed to the lambda function. The output of the lambda function is stored in the `response` field. You can view the output of the lambda function in the `Output` tab of the state machine. You can also view the execution history of the state machine in the `Execution details` tab. 

A successful execution of the state machine will look like this:
<img width="1417" alt="Screenshot 2024-04-22 at 11 14 05 AM" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/bd8af7de-0b15-413b-93d2-11415fead6a5">


### Caveats
In testing the state machine, I encountered a few issues that I would like to highlight:

1. **Role Permissions**: Make sure that the role you provide to the state machine has the necessary permissions to execute the lambda function. I tried to use an old IAM role for this step function, however, this rendered the following error:
```
{
  "Error": "Lambda.AWSLambdaException",
  "Cause": "User: arn:aws:sts::905418434745:assumed-role/StepFunctions-Individual_Project4-role-ymee4vnsn/pUBglYSipyAbyPsehEoWHmOFMDviUKzd is not authorized to perform: lambda:InvokeFunction on resource: arn:aws:lambda:us-east-1:905418434745:function:Rust-Serverless-Microservice-PutFunction-UAK78Ex7vK1x:$LATEST because no identity-based policy allows the lambda:InvokeFunction action (Service: Lambda, Status Code: 403, Request ID: ac614757-68d3-4bc8-9c7d-cc991c0dafa2)"
}
```
With this output: 
<img width="1710" alt="Screenshot 2024-04-22 at 11 05 31 AM" src="https://github.com/AaryaDesai1/Rust_Microservice/assets/143753050/b9ad3cd3-df06-4736-95c7-fd1eed7db6a5">

For the same, I navigated back to my state machine and made sure to `Create a new role` and provide the necessary permissions to the role. This resolved the issue.

## Conclusion
And that's it! You have successfully created a step function that orchestrates the lambda functions created in the previous project. You can now trigger the step function using an API Gateway endpoint. This project was a great learning experience and I hope you found this tutorial helpful. If you have any questions or feedback, feel free to reach out to me.