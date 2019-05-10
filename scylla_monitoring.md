In this post, we're going to deploy Scylla monitoring on EC2 using the ECS AMI.

You can find Scylla monitoring documentation [here](https://docs.scylladb.com/operating-scylla/monitoring/monitoring_stack/).

Scylla monitoring uses docker containers.  Thus, the easiest way to get it up and running in the cloud is to use an Elastic Container Service (ECS) Amazon Machine Image (AMI), because it comes pre-baked with all the docker goodies.  To find the most current ECS AMI run the following command:

```

aws ssm get-parameters --names /aws/service/ecs/optimized-ami/amazon-linux-2/recommended --region us-east-1

```

You should see output similar to:

```
{
    "InvalidParameters": [],
    "Parameters": [
        {
            "Name": "/aws/service/ecs/optimized-ami/amazon-linux-2/recommended",
            "LastModifiedDate": 1555437076.109,
            "Value": "{\"schema_version\":1,\"image_name\":\"amzn2-ami-ecs-hvm-2.0.20190402-x86_64-ebs\",\"image_id\":\"ami-0bc08634af113cccb\",\"os\":\"Amazon Linux 2\",\"ecs_runtime_version\":\"Docker version 18.06.1-ce\",\"ecs_agent_version\":\"1.27.0\"}",
            "Version": 12,
            "Type": "String",
            "ARN": "arn:aws:ssm:us-east-1::parameter/aws/service/ecs/optimized-ami/amazon-linux-2/recommended"
        }
    ]
}
```

Copy the `image_name` (i.e. `amzn2-ami-ecs-hvm-2.0.20190402-x86_64-ebs`) and head over to the AWS console.  Because this is just for monitoring, we'll skip the Auto-Scaling Group and Launch Configuration and just launch the box using [the wizard](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#LaunchInstanceWizard:).  Click on "Community AMIs" and paste the `image_name` into the search field.  This should yield one result, which you can then Select.

For instance type, we want at least 15GB RAM for our production instance, so we'll go with a `t2.xlarge`.  For storage, I left the boot volume alone and added an EBS volume with 1024GB of magnetic.  For the security group, I created a new one.  I left the universal access for ssh and added a custom TCP rule to allow access to port 3000 only from my IP.





