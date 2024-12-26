******

<details>
<summary>Exercise 1: Working with Subnets in AWS </summary>
 <br />

```python
import boto3

# Get all the subnets in your default region
# Print the subnet Ids

ec2_client = boto3.client('ec2', 'eu-central-1')

subnets = ec2_client.describe_subnets(
    Filters=[
        {
            'Name': 'vpc-id',
            'Values': [
                'vpc-060869f217db62513',
            ]
        },
    ],
)['Subnets']

subnet_ids = [subnet['SubnetId'] for subnet in subnets]

print(subnet_ids)
```

</details>

******

<details>
<summary>Exercise 2: Working with IAM in AWS </summary>
 <br />

```python
import boto3

# Get all the IAM users in your AWS account
# For each user, print out the name of the user and when they were last active (hint: Password Last Used attribute)
# Print out the user ID and name of the user who was active the most recently

iam_client = boto3.client('iam', region_name='eu-central-1')

users = iam_client.list_users()['Users']

last_active_users = [f"User {user['UserName']} was active at {user['PasswordLastUsed']}" if 'PasswordLastUsed' in user else f"User {user['UserName']} never was active"  for user in users]

# print("\n".join(last_active_users))
print(*last_active_users, sep='\n')

print('-----------------------')

sorted_by_password_last_used_users = sorted([user for user in users if 'PasswordLastUsed' in user], key=lambda u: u['PasswordLastUsed'], reverse=True)

most_recently_active_user = sorted_by_password_last_used_users[0]

print(f"Most recently active user is {most_recently_active_user['UserName']} who was active at {most_recently_active_user['PasswordLastUsed']}")
```

</details>

******

<details>
<summary>Exercise 3: Automate Running and Monitoring Application on EC2 instance </summary>
 <br />

```python
import time
import boto3
import paramiko
import requests
import schedule

# Start EC2 instance in default VPC
# Wait until the EC2 server is fully initialized
# Install Docker on the EC2 server
# Start nginx container
# Open port for nginx to be accessible from browser
# Create a scheduled function that sends a request to the nginx application and checks the status is OK
# If status is not OK 5 times in a row, it restarts the nginx application


ec2_resource = boto3.resource('ec2', region_name='eu-central-1')
ec2_client = boto3.client('ec2', region_name='eu-central-1')

def get_instance():
    # check for instance to not create it if exists
    instances = ec2_client.describe_instances(
        Filters=[
            {
                'Name': 'tag:Name',
                'Values': [
                    'myapp-server',
                ]
            },
        ]
    )

    instance_exists = len(instances['Reservations']) != 0 and len(instances['Reservations'][0]['Instances']) != 0

    if instance_exists:
        return instances['Reservations'][0]['Instances'][0]['InstanceId']

    image_id = 'ami-06902dc42c62c6ff2'
    key_name = 'myapp-key-pair'
    instance_type = 't2.small'

    created_instances = ec2_resource.create_instances(
        ImageId=image_id,
        KeyName=key_name,
        MinCount=1,
        MaxCount=1,
        InstanceType=instance_type,
        TagSpecifications=[
            {
                'ResourceType': 'instance',
                'Tags': [
                    {
                        'Key': 'Name',
                        'Value': 'myapp-server'
                    },
                ]
            },
        ]
    )
    return created_instances[0].id

instance_id = get_instance()

# Check Instance to be running.
instance_is_running = False
while not instance_is_running:
    instance_statuses = ec2_client.describe_instance_status(
        InstanceIds=[
            instance_id,
        ]
    )['InstanceStatuses'][0]

    instance_is_running = instance_statuses['InstanceStatus']['Status'] == 'ok' and instance_statuses['SystemStatus']['Status'] == 'ok' and instance_statuses['InstanceState']['Name'] == 'running'

    if not instance_is_running:
        print('Waiting for instance to be available...')
        time.sleep(2)

instances = ec2_client.describe_instances(
    Filters=[
        {
            'Name': 'tag:Name',
            'Values': [
                'myapp-server',
            ]
        },
    ]
)

instance = instances["Reservations"][0]["Instances"][0]

permissions = ec2_client.describe_security_groups(
    GroupNames=['default']
)['SecurityGroups'][0]['IpPermissions']

port_8080_open = False
port_22_open = False
for permission in permissions:
    print(permission)
    # some permissions don't have FromPort set
    if 'FromPort' in permission and permission['FromPort'] == 8080:
        port_8080_open = True
    if 'FromPort' in permission and permission['FromPort'] == 22:
        port_22_open = True


print(port_8080_open)
print(port_22_open)

if not port_8080_open:
    ec2_client.authorize_security_group_ingress(
        FromPort=8080,
        ToPort=8080,
        GroupName='default',
        CidrIp='0.0.0.0/0',
        IpProtocol='tcp'
    )

if not port_22_open:
    ec2_client.authorize_security_group_ingress(
        FromPort=22,
        ToPort=22,
        GroupName='default',
        CidrIp='95.91.243.73/32', # todo move in env variable
        IpProtocol='tcp'
    )

ssh_client = paramiko.SSHClient()
ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh_client.connect(hostname=instance["PublicIpAddress"], username='ec2-user', key_filename='myapp-key-pair.pem')

commands = [
    'sudo yum update -y && sudo yum install -y docker',
    'sudo systemctl start docker',
    'sudo usermod -aG docker ec2-user',
    'docker run -d -p 8080:80 --name nginx nginx',
]

for command in commands:
    stdin, stdout, stderr = ssh_client.exec_command(command)
    print(stdout.readlines())

ssh_client.close()

request_application_attempts = 0

def restart_server():
    ssh_client = paramiko.SSHClient()
    ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh_client.connect(hostname=instance["PublicIpAddress"], username='ec2-user', key_filename='myapp-key-pair.pem')

    stdin, stdout, stderr = ssh_client.exec_command('docker start nginx')
    print(stdout.readlines())

    ssh_client.close()

    global request_application_attempts
    request_application_attempts = 0

    return

def monitor_application():
    global request_application_attempts
    try:
        response = requests.get(f"{instance['PublicIpAddress']}")

        if response.status_code == 200:
            print("Application is running")
        else:
            print("Application is down. Fix it")
            request_application_attempts += 1

            if request_application_attempts == 5:
                restart_server()
    except Exception as error:
        print(f'Application not accessible at all: {error}')
        request_application_attempts += 1
        if request_application_attempts == 5:
            restart_server()
        return


schedule.every(30).seconds.do(monitor_application)

while True:
    schedule.run_pending()
```

</details>

******

<details>
<summary>Exercise 4: Working with ECR in AWS </summary>
 <br />

```python
import boto3

ecr_client = boto3.client('ecr', region_name='eu-central-1')

# Get all the repositories in ECR
# Print the name of each repository
# Choose one specific repository, and for that repository list all the image tags inside, sorted by date, with the most recent image tag on top

repositories = ecr_client.describe_repositories()['repositories']

repository_names = [repository['repositoryName'] for repository in repositories]

print(f"Repository names: {repository_names}")

chosen_repository = repositories[0]
repository_name = chosen_repository['repositoryName']

images = ecr_client.describe_images(repositoryName=repository_name)['imageDetails']

sorted_images = sorted(images, key=lambda x: x['imagePushedAt'], reverse=True)

for image in sorted_images:
    print({'tag': image['imageTags'], 'pushed_at': image['imagePushedAt']})
```

</details>

******

<details>
<summary>Exercise 5: Python in Jenkins Pipeline </summary>
 <br />

### Start jenkins

`jenkins-agent`
```dockerfile
FROM jenkins/ssh-agent:alpine

RUN apk update
RUN apk add docker docker-cli-compose git curl

RUN apk add --no-cache --repository=http://dl-cdn.alpinelinux.org/alpine/v3.20/main/ nodejs=20.15.1-r0 npm

# Set environment variables
ENV VIRTUAL_ENV=/opt/venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Install Python and tools, then set up the virtual environment
RUN apk add --no-cache python3 py3-pip \
    && python3 -m venv $VIRTUAL_ENV \
    && $VIRTUAL_ENV/bin/pip install --upgrade pip \
    && $VIRTUAL_ENV/bin/pip install boto3 paramiko requests

RUN git config --global user.email "jenkins@example.com"
RUN git config --global user.name "jenkins"
```

```shell
cd jenkins
sh run_jenkins_with_docker.sh
```

### Configure jenkins agent

Add `jenkins` credentials
```text
Manage Jenkins -> Manage Credentials -> Add Credentials. Create credentials with

Kind: SSH Username with private key
id: jenkins
username: jenkins
Private Key: Enter directly -> on host `cat ~/.ssh/jenkins_agent_key`
```

Add new node
```text
Manage Jenkins -> Manage Nodes and clouds -> New Node with

Remote root directory: /home/jenkins
label: agent1
usage: only build jobs with label expression
Launch method: Launch agents by SSH
Host -> `docker container inspect agent1 | grep IPAddress`
Credentials: jenkins
Host Key verification Strategy: Manually trusted key verification
```

### Push docker image to ECR
```shell
(venv) (base) ➜  Exercises export AWS_DEFAULT_PROFILE=paypolitan
(venv) (base) ➜  Exercises aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 173861077519.dkr.ecr.eu-central-1.amazonaws.com
(venv) (base) ➜  Exercises docker build -t myapp .
(venv) (base) ➜  Exercises docker tag myapp:latest 173861077519.dkr.ecr.eu-central-1.amazonaws.com/myapp:1.0
(venv) (base) ➜  Exercises docker tag myapp:latest 173861077519.dkr.ecr.eu-central-1.amazonaws.com/myapp:2.0
(venv) (base) ➜  Exercises docker tag myapp:latest 173861077519.dkr.ecr.eu-central-1.amazonaws.com/myapp:3.0

(venv) (base) ➜  Exercises docker push 173861077519.dkr.ecr.eu-central-1.amazonaws.com/myapp:1.0
(venv) (base) ➜  Exercises docker push 173861077519.dkr.ecr.eu-central-1.amazonaws.com/myapp:2.0
(venv) (base) ➜  Exercises docker push 173861077519.dkr.ecr.eu-central-1.amazonaws.com/myapp:3.0
```

### Create Pipeline

#### Create jenkins credentials
    From `cat ~/.aws/credentials`:
       Create `jenkins_aws_access_key_id` credentials
       Create `jenkins_aws_secret_access_key` credentials
    (Secret text) in Jenkins

#### Create Pipeline
    Name: deploy-to-jenkins
    Configuration
        Description: Create a Jenkins job that fetches all the available images from your application's ECR repository using Python. It allows the user to select the image from the list through user input and deploys the selected image to the EC2 server using Python.
        Do not allow concurrent builds
            Abort previous builds
        Throttle builds
            Number of builds: 1
            Time period: Minute

### Create AWS SSM `myapp-rsakey` Parameter (will use it in a deploy script)

Manually create `myapp-rsakey` SecureString parameter in AWS SSM Parameter Store from `myapp-key-pair.pem`
```shell
(venv) (base) ➜  Exercises cat myapp-key-pair.pem 
-----BEGIN RSA PRIVATE KEY-----
...
...
...
-----END RSA PRIVATE KEY-----
```

### Write python scripts

`fetch_images.py`
```python
import boto3
import os

ecr_repository_name = os.environ['ECR_REPOSITORY_NAME']
region_name = os.environ['REGION_NAME']

ecr_client = boto3.client('ecr', region_name=region_name)

# Fetch all 3 images from the ECR repository
images = ecr_client.describe_images(repositoryName=ecr_repository_name)

image_tags = [tag for tag in images['imageDetails'][0]['imageTags']]

for tag in image_tags:
    print(tag)
```

`deploy.py`
```python
import base64
import os
from io import StringIO

import boto3
import paramiko

# get all the env vars set in Jenkinsfile

region_name = os.environ['REGION_NAME']
ssh_host = os.environ['EC2_SERVER']

docker_registry = os.environ['ECR_REGISTRY']

# version is selected by user in Jenkins
docker_image = os.environ['DOCKER_IMAGE']

# Get Parameter from SSM
# @param - string | list comma seperated
def get_parameters(param):
    # Boot up SSM client
    ssm = boto3.client('ssm', region_name=region_name)

    # Retrieve SSM from Parameter Store
    response = ssm.get_parameters(
        Names=[param], WithDecryption=True
    )

    for parameter in response['Parameters']:
        return parameter['Value']


# fetch authorization token using boto3 ecr client interface
ecr = boto3.client('ecr', region_name=region_name)
response = ecr.get_authorization_token()['authorizationData'][0]
docker_pwd = base64.b64decode(response['authorizationToken']).decode('utf-8').split(':')[1]

# Get RSA Key parameter store
pkey = paramiko.RSAKey.from_private_key(file_obj=StringIO(get_parameters('myapp-rsakey')))

# SSH into the EC2 server
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname=ssh_host, port=22, username='ec2-user', pkey=pkey, timeout=10)

stdin, stdout, stderr = ssh.exec_command(f"echo {docker_pwd} | docker login {docker_registry} --username AWS --password-stdin")
print(stdout.readlines())
print(stderr.readlines())

stdin, stdout, stderr = ssh.exec_command(f"docker ps -q --filter \"publish=8080\" | xargs -r docker stop")
print(stdout.readlines())
print(stderr.readlines())

stdin, stdout, stderr = ssh.exec_command(f"docker run -p 8080:8080 -d --rm --name=myapp {docker_image}")
print(stdout.readlines())
print(stderr.readlines())

ssh.close()
```

`check_application.py`
```python
import requests
import time
import os

ssh_host = os.environ['EC2_SERVER']

# Validate that the application was successfully started and is accessible by sending a request to the application
try:
    # give the app some time to start up
    time.sleep(15)

    response = requests.get(f"http://{ssh_host}:8080")
    if response.status_code == 200:
        print('Application is running successfully!')
    else:
        print('Application deployment was not successful')
except Exception as ex:
    print(f'Connection error happened: {ex}')
    print('Application not accessible at all')
```

### Upload python scripts on jenkins agent server
```shell
(venv) (base) ➜  task5 docker cp ./fetch_images.py agent1:/home/jenkins/workspace/deploy-to-ec2/fetch_images.py
                                             Successfully copied 2.05kB to agent1:/home/jenkins/workspace/deploy-to-ec2/fetch_images.py

(venv) (base) ➜  task5 docker cp ./deploy.py agent1:/home/jenkins/workspace/deploy-to-ec2/deploy.py 
                                             Successfully copied 3.07kB to agent1:/home/jenkins/workspace/deploy-to-ec2/deploy.py
                                             
(venv) (base) ➜  task5 docker cp ./check_application.py agent1:/home/jenkins/workspace/deploy-to-ec2/check_application.py
                                             Successfully copied 2.56kB to agent1:/home/jenkins/workspace/deploy-to-ec2/check_application.py
```

### Update pipeline configuration

#### [v] This project is parameterized
```
    ECR_REGISTRY: 173861077519.dkr.ecr.eu-central-1.amazonaws.com
    ECR_REPOSITORY_NAME: myapp
    REGION_NAME: eu-central-1
    EC2_SERVER: 3.66.166.129
```

#### Configure pipeline
    Definition: Pipeline script

`Script`
```text
#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        ECR_REGISTRY = "${params.ECR_REGISTRY}"
        ECR_REPOSITORY_NAME = "${params.ECR_REPOSITORY_NAME}"

        REGION_NAME = "${params.REGION_NAME}"
        EC2_SERVER = "${params.EC2_SERVER}"

        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
    }
    stages {
        stage('select image version') {
            steps {
              script {
                  echo 'fetching available image versions'
                  def result = sh(script: '/opt/venv/bin/python3 fetch_images.py', returnStdout: true).trim()
                  // split returns an Array, but choices expects either a List or String, so we do "as List"
                  def tags = result.split('\n') as List
                  version_to_deploy = input message: 'Select image version', ok: 'Deploy', parameters: [choice(name: 'Select image', choices: tags)]
                  // put together the full image name
                  env.DOCKER_IMAGE = "${ECR_REGISTRY}/${ECR_REPOSITORY_NAME}:${version_to_deploy}"
                  echo env.DOCKER_IMAGE
              }
            }
        }
        stage('deploying image') {
            steps {
                script {
                   echo 'deploying docker image to EC2...'
                   def result = sh(script: '/opt/venv/bin/python3 deploy.py', returnStdout: true).trim()
                   echo result
                }
            }
        }
        stage('validate deployment') {
            steps {
                script {
                   echo 'validating that the application was deployed successfully...'
                   def result = sh(script: '/opt/venv/bin/python3 validate.py', returnStdout: true).trim()
                   echo result
                }
            }
        }
    }
}
```
</details>

******
