create a linux ec2 instance
sudo yum install docker
sudo yum install git
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker
Clone your repo (E-commerce)
ls
do ls -al
cd into ecommerce-app-source-code
ls
cd product
do ls
	*** Docker Images Creation ***
# Replace hodalo by your docker registry username
sudo docker build -t hodalo/productapp:latest .
switch to root (sudo su)
- then docker images (to see the newly build image)
- cd ../paymentapp/
- ls
- docker build -t hodalo/paymentapp:latest .
- docker images
- cd ../frontendapp/
- ls
- docker build -t hodalo/frontendapp:latest .
- docker images
(ignore the error message)
# check your CLI has been configured
aws s3 ls
# If not
aws configure
	*** IAM Role Creation ***
				
- Navigate to IAM
- Create roles
- Select AWS service
- Use case: ec2 role for elastic container service (last option)
- Next, Next
- Role name: ecsInstanceRole
- Create role
	*** CloudFormation Stacks Creation ***
	
1- Navigate to your Github and Copy the content of create_IAM_roles.yml from ecs-project-cluster-setup/1.create_IAM_roles
Navigate back to CloudFormation and create a stack named ecs-project-iam-roles
2- Navigate to your Github and Copy the content of core-infrastructure-setup.yml from ecs-project-cluster-setup/2.Core-infrastructure-setup
Navigate back to CloudFormation and create a stack named ecs-project-infrastructure-setup
3- Navigate to your Github and Copy the content of alb-external.yml from ecs-project-cluster-setup/3.loadbalancer/ALB
Navigate back to CloudFormation and create a stack named ecs-project-loadbalancer
4- Navigate to your Github and Copy the content of ecs-ec2-via-cloudformation.yml from Prof's ecs-project-cluster-setup/CloudFormation-based
Navigate back to CloudFormation and create a stack named ecs-project-cluster-env
- Navigate to IAM
- IAM roles
- Search and open ecsTaskExecution  role
- Copy role's ARN
- Navigate and open the Ecommerce-app-deploy-project-ecs repository on your local (vscode)
- Navigate and open the microservices-td
- Update the 3 files names to have <td-frontendapp-setup.json>, <td-paymentapp-setup.json>, and <td-productapp-setup.json>
- In the td-productapp-setup.json:
	. replace the role's ARN copied by the one on line 2 and 3
	. line 13: "image": "pauloclouddev/flamencoapp:latest",
	. line 22: "name": "productapp"
	. line 25: "family": "td-productapp",
- In the td-paymentapp-setup.json:
	. replace the role's ARN copied by the one on line 2 and 3
	. line 13: "image": "pauloclouddev/operaapp:latest",
	. line 22: "name": "paymentapp"
	. line 25: "family": "td-paymentapp",
- In the td-frontendapp-setup.json:
	. replace the role's ARN copied by the one on line 2 and 3
	. line 21: "image": "pauloclouddev/musicboxapp:latest",
	. line 31: "name": "frontendapp"
	. line 34: "family": "td-frontendapp",
- cd Ecommerce-app-deploy-project-ecs/
# run these commands from the README.md file by replacing this region with your default region
- aws ecs register-task-definition  --cli-input-json file://td-productapp-setup.json --region us-east-1
- aws ecs register-task-definition  --cli-input-json file://td-paymentapp-setup.json --region us-east-1
- aws ecs register-task-definition  --cli-input-json file://td-frontendapp-setup.json --region us-east-1
	*** Create an ECS Service ***
	
- Navigate to ECS
- Open the  ecs-project-ec2
- Create service
- Launch type: EC2
- Task Definition: td-productapp
- Service Name: productapp-service
- Number of tasks: 1
- Leave everything else default
- Next
- VPC: choose the one with CICR 172.16.0.0/16
- sUBNETS: BOTH
- Security group: Edit
	. name: productapp-service-sg
	. type: Custom TCP, pORT 9001 to Anywhere
- Next, Next, Create service, View service
- Once running, copy Private IP Id
	*** Create a task definition ***
- Task Definition
- Open frontendapp
- select latest revision
- create new revision
- scroll all the way down and click the container name (productapp-service)
- Replace the <product_host> place holder with the Private IP copied to have something like 172.0.0.0:9001
REPEAT STEPS TO CREATE  paymentapp-service (port 9002) with its task definition revisionon, frontendapp-service (port 9000) with its task definition revision.