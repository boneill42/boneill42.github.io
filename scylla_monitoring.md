
Scylla monitoring leverages docker containers:

https://docs.scylladb.com/operating-scylla/monitoring/monitoring_stack/



Thus, the easiest way to get it up and running in the cloud is to use an Elastic Container Service (ECS) Amazon Machine Image (AMI), because it comes pre-baked with all the docker goodies.  To find the most current ECS AMI run the following command:



```

aws ssm get-parameters --names /aws/service/ecs/optimized-ami/amazon-linux-2/recommended --region us-east-1

```


