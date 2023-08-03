pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID="401277835798"
        AWS_DEFAULT_REGION="us-east-2"
        IMAGE_REPO_NAME="jenkins-pipeline"
        IMAGE_TAG= "${env.BUILD_ID}"
        REPOSITORY_URI = "401277835798.dkr.ecr.us-east-2.amazonaws.com/jenkins-pipeline"
    }
    stages {
        stage('Git checkout') {
            steps {
                git 'https://github.com/ugbebor18/realone-repo-kube.git'
            }
        }
        
        stage('Build with maven') {
            steps {
                sh 'cd SampleWebApp && mvn clean install'
            }
        }
        
             stage('Test') {
            steps {
                sh 'cd SampleWebApp && mvn test'
            }
        
            }
        
        
         stage('Logging into AWS ECR') {
                     environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
                         
                   }
                     steps {
                       script{
             
                         sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                }
                 
            }
        }
      
          stage('Building image') {
            steps{
              script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
        
        stage('Pushing to ECR') {
          steps{  
            script {
                sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
         }
        }
      }
         
         stage('pull image & Deploying application on eks cluster') {
                    environment {
                       AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                       AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                 }
                    steps {
                      script{
                        dir('kubernetes/') {
                          sh 'aws eks update-kubeconfig --name myapp-eks-cluster --region us-east-2'
                          sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                          sh 'helm upgrade --install --set image.repository="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}" --set image.tag="${VERSION}" myjavaapp myapp/' 



 
                        }
                    }
               }
            }
    }
}
