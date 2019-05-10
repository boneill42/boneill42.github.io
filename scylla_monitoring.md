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

For instance type, we want at least 15GB RAM for our production instance, so we'll go with a `t2.xlarge`.  For storage, I left the boot volume alone and added an EBS volume with 1024GB of magnetic.  For the security group, I created a new one.  Because I'm paranoid, I lock down the ssh access to only my IP.  And I don't configure any external access to port 3000, on which monitoring runs.  If you are less paranoid, go ahead and allow access to ssh from anywhere and port 3000 from your IP.


Launch the instance and wait for it to come up.

Now, we need to attach the EBS volume.  You can follow [this documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html), but assuming this is a new volume, it should be as simple as:
```
sudo mkfs -t xfs /dev/xvdb
```

Then, we mount it and make it available for writing:
```
sudo mkdir /data
sudo mount /dev/xvdb /data
sudo chown ec2-user /data
```

At this point, we're ready to install Scylla monitoring.  Check for the [latest release](https://github.com/scylladb/scylla-grafana-monitoring/releases).  Then ssh to the new box.

Grab scylla monitoring with the following command (replacing X with the latest version number):
```
sudo yum install -y wget
wget https://github.com/scylladb/scylla-grafana-monitoring/archive/scylla-monitoring-2.X.tar.gz
tar -xvf scylla-monitoring-2.X.tar.gz
cd scylla-grafana-monitoring-scylla-monitoring-2.X
```

You may want to verify connectivity to your scylla servers.  The monitoring tool will need access to port 9180.  You can do the following:
```
sudo yum install -y telnet
telnet 172.42.5.55 9180
```

If you have connectivity, you should see:
```
Escape character is '^]'.
^]quit
```

After verifying connectivity to your database hosts, update the following config file with the host information: `prometheus/scylla_servers.yml`.

e.g.
```
# List Scylla end points
- targets:
       - 172.42.5.1:9180
       - 172.42.6.2:9180
       - 172.42.7.3:9180

  labels:
       cluster: DevScyllaCluster
       dc: us-east
```

Now, fire her up with:
```
./start-all.sh -s prometheus/scylla_servers.yml -d /data
```
You should see something like:
```
...
Wait for Grafana container to start..
Start completed successfully, check http://localhost:3000
```

And verify that it is running with:
```
telnet localhost 3000
```

If you were less paranoid before and opened up access, you should be able to hit http://YOUR_INSTANCE_IP:3000 and see the monitoring console.

If you were paranoid before then you'll need to setup port forwarding so you can get to the monitoring port.  Run the following command _on your local machine_.
```
ssh -L 3000:localhost:3000 ec2-user@your_instances_public_ip
```

You should get a command prompt on the instance, which means port-forwarding has been setup.  Now, just go to http://localhost:3000 in a browser, and you should be all set!

Happy monitoring!




