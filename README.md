# phoenix

phoenix is a fun "proof of concept" side project designed to automatically trigger EC2 instances for Ethereum mining. Instances will automatially spin up or down in response to currnt Ethereum mining profitability. This repo contains a CloudFormation stack YAML file for building the infrastructure in AWS.

## What was the goal of this project?

The goal was mainly to develop something using multiple AWS resources and build some CloudFormation experience, while also having a little fun. AWS GPU instances are not cheap to operate, so currently the project fluctuates in and out of profitability and spends most of the time dormant. However occasionally I will get a notification that instances are spinning up, and it's fun to see.

## How It Works

Ethereum mining requires GPU processing power, so we'll need to determine the best EC2 GPU instance to use. AWS offers some really powerful GPUs, but they come at a hefty price. After measuring the mining hashrates of each GPU instance against their hourly cost, we see that the g4dn.xlarge offers the best mining performance for the price, so this is the only instance we'll use for now. To keep operating costs as low as possible we'll be using Spot Pricing to reduce costs by 70%!

*Note: You may need submit a request to AWS for a quota increase in "G-Spot Instances", as the default allowed is 0*

When you create the stack, it will prompt you to enter your Ethereum wallet address, email (optional) for AutoScaling notifications, max number of instances, and the VPC and subnets to which this stack should be deployed. The stack will create all of the necessary resources and IAM roles.

![alt text](https://benwaddell.s3.amazonaws.com/github/phoenix/stackdetails.png)

Once the stack is created, a Lambda function will run every 3 minutes to get the current profitability of Ethereum mining. This is done by using the WhatToMine.com API to calculate the current rewards after deducting the AWS operating costs.

If the rewards outweigh the costs, the Lambda function will adjust the AutoScaling Group (ASG) to increase the minimum instance capacity. This will immediately trigger the ASG to create a new EC2 instance. The instance uses a "Deep Learning AMI" that is pre-configured with the necessary graphics drivers. The instance will download and extract PhoenixMiner, create a startup script to launch PhoenixMiner with your Ethereum wallet address, and then run the script.

Lambda will continue to check profitability every 3 minutes and gradually increase the minimum capacity as the profitability *increases*, until it reaches the "Max Number of Instances" specified during stack creation. As the minimum capacity increases, new instances will be deployed. If profitability *decreases*, Lambda will gradually reduce the ASG minimum instance capacity, which will begin terminating instances. Instances will not terminate until they have run for at least one hour, to prevent unnecessary frequent creations and terminations if the profitability bounces around. With each instance creation or termination, a notification will be sent, if you supplied your email address during stack creation.

The entire process is fully automated and you can "set it and forget it". Aside from the EC2 instances, all other resources are free to operate, so leaving the system in place should not cost anything until an instance is created, which only happens if it becomes profitable to do so.


#### Diagram:
![alt text](https://benwaddell.s3.amazonaws.com/github/phoenix/phoenixtemplate.png)

## This is just free money, right?

No, not exactly. Mining profits fluctuate depending on transaction fees (these get paid to the miners) and the underlying value of Ethereum. Even with Ethereum prices currently at all time highs, the costs of operating mining servers in AWS is rarely profitable. Additionally, the mining difficulty is always increasing, which continues to lower rewards over time. Occasionally this can actually be profitable, but unless the mining rewards begin to consistently outweigh the costs, it is not a great investment.

Some argue that this is a poor use of AWS resources. I don't necessarily disagree, but since we are only using servers that are otherwise inactive (through Spot Pricing), we are not really taking away resources that are in demand elsewhere.

Overall, this is not intended to be a "get rich" plan, but instead a way to have some fun playing around in AWS, while also earning some rewards in the process.





