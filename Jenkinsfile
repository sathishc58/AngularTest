def dockerImage
def buildNumber = currentBuild.number

parameters {
        gitParameter branchFilter: 'origin/(.*)',
                     name: 'branchName', 
                     type: 'PT_BRANCH',
                     defaultValue: 'master'
}

pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION="us-east-1"
        AWS_ACCOUNT_ID="807347334860"
        IMAGE_REPO_NAME="testrepo"
        IMAGE_TAG="21"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs();
            }
        }
        stage('Cloning Repo') {
            steps {
                print "Build numebr is " +  buildNumber
                git branch: "${params.branchName}",
                  credentialsId: '073c22ea-f05b-409d-99a4-5ace933cb7ab',
                  url:  'https://github.com/sathishc58/AngularTest.git'
            }
        }
        stage('Login to ECR') {
            steps {
                print "Creating Docker Image ... Please wait"
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
  
            }
        }
        stage('Create docker image and push') {
            steps {
                script {
                       dockerImage = docker.build "${IMAGE_REPO_NAME}:${BUILD_NUMBER}" 
                }
            }
        }
        stage('Push docker image') {
            steps {
                script {
                     sh "docker tag ${IMAGE_REPO_NAME}:${BUILD_NUMBER} ${REPOSITORY_URI}:$BUILD_NUMBER"
                     sh "docker tag ${IMAGE_REPO_NAME}:${BUILD_NUMBER} ${REPOSITORY_URI}:latest"
                     sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${BUILD_NUMBER}"
                     sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:latest"
                }
            }
        }
        /* stage('Copy deployment manifest') {
            steps {
                sh """cp /tmp/nginx.json ${WORKSPACE}/nginx.json
                  chown jenkins:jenkins ${WORKSPACE}/nginx.json
                  chmod 755 ${WORKSPACE}/nginx.json"""
            }
        }
        stage('Deploy to ECS') {
            steps {
                
                sh """aws ecs register-task-definition --cli-input-json file://${WORKSPACE}/nginx.json
                aws ecs list-task-definitions
                aws ecs run-task --cluster default --task-definition sample:9 --count 1"""
            }
        } */
    }
}
