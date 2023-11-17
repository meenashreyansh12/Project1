
# Build CI / CD Pipeline using Jenkins and deploy the real world Web Application in AWS Cloud

Technologies Used:-
-------------------
1. Jenkins
2. Groovy
3. AWS Cloud
4. Git
5. Docker  

Pre-Requiset:
--------------
* Setup Jenkins (ubuntu with t2.large)
* Setup webhook integration for automatic Build Triggers 
* Create A Role (name: meen_new_role) with AmazonEC2ContainerRegistryFullAccess and attach to Jenkins machine  
* Create ECR Repository in the same region where jenkins machine is available to upload image into that repository

JenkinsFile Code steps
-------------------------------
>step1: To build pipeline code, we need to create an agent machine and include here to build the job from agent machine 
   
    pipeline
    {
        agent any
    }

>step2: To Pass environments in jenkins

    environment {
        APP_NAME = "myapp"
        ECR_REGISTRY = "115498080659.dkr.ecr.us-east-1.amazonaws.com"
        ECR_REPOSITORY = "thej"
        IMAGE_NAME = "${APP_NAME}"
        BUILD_VERSION = getVersion()
    }

>step3: Download the code from repository

        stage("Download Code")
        {
            steps
            {
                git 'https://github.'
            }
        }

>step4: Build and test application code using maven 

        stage("Test Application")
        {
            steps
            {
                    sh 'mvn test'
            }
        }
        stage("Build Application Code")
        {
            steps
            {
                sh 'mvn package'
            }
        }

>step5: Build the Docker image 
    
* First download Docker in Jenkins machine for building image from Dockerfile
       
        # sudo curl -fsSL https://get.docker.com -o install-docker.sh
        # sudo sh install-docker.sh

* Add jenkins user in docker group to build docker images directly 
        # sudo usermod -a -G docker jenkins
  
* Add jenkins user in sudo group to run root commands. Also, enter details of jenkins user in sudoers file to avoid asking for password [ jenkins ALL (ALL:ALL) NOPASSWD:ALL ]
       
        # sudo usermod -a -G sudo jenkins
        # vim /etc/sudoers
        
* To Build Docker image    
        
        stage("Build Docker image")
        {
            steps
            {
                 sh "docker build -t $APP_NAME:$BUILD_VERSION ."
            }
        }

>step6: To push Docker image into ECR repository 
* If Jenkins machine is avilable in AWS cloud environment, that machine need to contain a role with AmazonEC2ContainerRegistryFullAccess and attach to Jenkins 
  machine 
* If Jenkins machine is available in local environment, then that machine requires a user () with AdministrationAccess  
* Check if ECR repository is avialabile, and if it's not available then create the repository in AWS ECR service 
* There are a group of steps included below to push the existing docker image to ECR repository
* Below is the jenkins code to upload image into ECR repository 

        stage("Push docker image to ECR")
        {
            steps
            {
              script
              {
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    sh "docker tag $APP_NAME:$BUILD_VERSION $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION"
                    sh "docker push $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION"
              }
            }
        }


>step7: Now pull the ECR image to local machine. For this, we need to login to ECR and deploy it into EC2 machine(docker)

* Setup Docker machine (specify security group in the inbound rules. Open ssh 22 port to admin access only and specify customised port 9090 to anyone to access our webpage)
* Establish the password-less authentication from Jenkins machine to Docker machine 
* Add ubuntu user in docker group to run the docker commands 

* For this we need to create a role with AmazonEC2ContainerRegistryFullAccess and attach it to the EC2-instance

* Authenticate first To aws ecr For this machine need ECR permissions 

        # aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}"

* Pull image from ECR    

        # docker pull $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION

>step10: stop old contaiers if running with same name (myappcontainer)

        stage("stop previuos containers") {
            steps{
                script{ 
                    sh 'ssh ubuntu@54.234.168.231 docker rm -f myappcontainer'
                }
            }
        }

>step11: Now Run container 

        stage("Run Docker image") {
            steps{
                sh "ssh ubuntu@54.234.168.231 docker run --name myappcontainer -d -p 9090:8080 $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION"
            }
        }


>step12: setup To send Notification To gmail ( for this we need to setup smtp port in jenkins to perticular gmail )

* according below steps if all steps gets success it will send project success message or if any step gets fail it will failed project
* To setup mail configuration in jenkins follw this link: https://drive.google.com/file/d/1G2HGfoGKyv3pzB1eLnW8mVxaUeltqpZ1/view?usp=drive_link
 

        post {
            success {
                mail bcc: '', body: 'ci-cd gets success', cc: '', from: '', replyTo: '', subject: 'Pojects completed successfully', to: 'goudc423@gmail.com'
                echo "Project completed Successfully"
            }
            failure {
                mail bcc: '', body: 'failed ci-cd', cc: '', from: '', replyTo: '', subject: 'Pojects Failed', to: 'goudc423@gmail.com'
                echo "Project Failed"
            }
        }


>step13: Specify the function to trigger build versions images 

        def getVersion() {
            def buildNumber = env.BUILD_NUMBER ?: '0'
            return "1.0.${buildNumber}"
        }

# All stages 

![Alt text](images/image.png)

# Gmail Notification 

![Alt text](images/image-1.png)
