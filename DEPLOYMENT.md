# Deployment Evidence

Fill this in as you go. Paste real output, not descriptions of output. A TA reads this
file with you at recitation.

## 1. Deployed URL and instance id

<!-- The ServiceUrl and InstanceId outputs. Paste both here every time
describe-stacks prints them, for the healthy deploy and for scenario 2. Both
change on every recreate, and you will need them for curls and sessions. -->
-------------------------------------------------------------------------
|                            DescribeStacks                             |
+------------+----------------------------------------------------------+
|  InstanceId|  i-0ea3add9a747a4c0d                                     |
|  ServiceUrl|  http://ec2-54-89-161-208.compute-1.amazonaws.com:8080   |
+------------+----------------------------------------------------------+

-----------------------------------------------------------------------
|                           DescribeStacks                            |
+------------+--------------------------------------------------------+
|  InstanceId|  i-059517e3626dabf10                                   |
|  ServiceUrl|  http://ec2-3-80-115-63.compute-1.amazonaws.com:8080   |
+------------+--------------------------------------------------------+

--------------------------------------------------------------------------
|                             DescribeStacks                             |
+------------+-----------------------------------------------------------+
|  InstanceId|  i-028501a76511a54bd                                      |
|  ServiceUrl|  http://ec2-54-242-188-217.compute-1.amazonaws.com:8080   |
+------------+-----------------------------------------------------------+

## 2. External health check

Run the check from your own machine, not from the instance. Paste the command and the
response.

```
✗ curl http://ec2-54-89-161-208.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}%   
```

## 3. What the template created

Three or four sentences, your own words. What compute, what network access, and what
glue made the service start.

The template created a `t3.micro` EC2 instance running Amazon Linux 2023. It
attached a security group that allows public TCP access to the service port
(8080 by default) and SSH port 22. The instance's user-data script installs and
starts Docker, then launches the lab service container with the configured port.

## 4. Scenario 2 diagnosis

**The failing curl** (command and output):

```
✗ curl http://ec2-3-80-115-63.compute-1.amazonaws.com:8080/api/health 
curl: (7) Failed to connect to ec2-3-80-115-63.compute-1.amazonaws.com port 8080 after 98 ms: Couldn't connect to server
```

**The log line that told you what was wrong:**

```
sh-5.2$ sudo docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
b8265c99dd81   ghcr.io/cmu-17-214/lab04-service:latest   "/__cacert_entrypoin…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   lab04-service
sh-5.2$ sudo docker logs lab04-service
lab04-service listening on 9090
```

**What was wrong, and the fix you applied:**

<!-- One or two sentences. Say what you changed and where you changed it. -->
The infra/params-scenario2.json changed the service port with PortOverride, but. docker still forward the 8080 to the old service port. I changed the port forwarding from 8080 to new effective port.

**The healthy curl after the fix:**

```
✗ curl http://ec2-54-242-188-217.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}%  
```

## 5. Teardown proof

Paste the delete output, or describe the console evidence that the resources are gone.

```
✗ aws cloudformation delete-stack --stack-name lab04-service  
aws cloudformation wait stack-delete-complete --stack-name lab04-service

✗ aws cloudformation describe-stacks --stack-name lab04-service

aws: [ERROR]: An error occurred (ValidationError) when calling the DescribeStacks operation: Stack with id lab04-service does not exist
```
