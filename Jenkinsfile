pipeline
{
    agent any
    
    environment
    {
        APP_NAME = "mytomee"
        ECR_REGISTRY = "433609462612.dkr.ecr.eu-west-2.amazonaws.com"
        ECR_REPOSITORY = "myimagerepo"
        IMAGE_NAME = "${APP_NAME}"
        BUILD_VERSION = getVersion()
    }
    stages
    {
        stage ('download the code')
        {
            steps
            {
                git 'https://github.com/meenashreyansh12/Project1.git'
            }
        }
        stage ('Test application')
        {
            steps
            {
                sh 'mvn test'
            }
        }
        stage ('Build application')
        {
            steps
            {
                sh 'mvn package'
            }
        }
        stage (' Build docker image')
        {
            steps
            {
                sh "docker build -t $APP_NAME:$BUILD_VERSION ."
            }
        }
        stage ('push docker image to ECR')
        {
            steps
            {
                script
                {
                    sh 'aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                    sh 'docker tag $APP_NAME $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION'
                    sh 'docker push $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION'
                }
            }
        }
        stage("Deploying ECR image") 
        {
            steps
            { 
                script
                { 
                    sh 'ssh ubuntu@172.31.7.122 aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                    sh 'ssh ubuntu@172.31.7.122 docker pull $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION'
                }
            }
        }
        stage("stop previuos containers") 
        {
            steps
            {
                    sh 'ssh ubuntu@172.31.7.122 docker rm -f myappcontainer'
            }
        }
        
        stage("Run Docker image")
        {
            steps
            {
                sh "ssh ubuntu@172.31.7.122 docker run --name myappcontainer -d -p 9090:8080 $ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_VERSION"
            }
        }
        stage("delete the previous container") 
        {
            steps
            {
                sh 'ssh ubuntu@172.31.7.122 docker rm -f myappcontainer'
            }
        }
        
    }
    post
        {
          success 
          {
            mail bcc: '', body: 'CI-CD project completed successfully', cc: '', from: '', replyTo: '', subject: 'Project completed successfully', to: 'meenakshishreyansh@gmail.com'
          }
          failure 
          {
            mail bcc: '', body: 'CI-CD project failed', cc: '', from: '', replyTo: '', subject: 'Project failed', to: 'meenakshishreyansh@gmail.com'
          }
       }
}
def getVersion() 
{
    def buildNumber = env.BUILD_NUMBER ?: '0'
    return "1.0.${buildNumber}"
}
