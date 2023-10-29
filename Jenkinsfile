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
                print "Build number is " +  buildNumber
                script {
                    echo "Git parameter is ${params.branchName}"
                    // git branch: "${params.branchName}", credentialsId: '72f13a10-4b6e-43c0-91b8-d6c0395687bf', url: 'https://github.com/sathishc58/AngularTest.git'
                    git branch: "${params.branchName}".split('/')[1], credentialsId: '72f13a10-4b6e-43c0-91b8-d6c0395687bf', url: 'https://github.com/sathishc58/AngularTest.git'
                }
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
        stage('Copy deployment manifest') {
            steps {
                sh """cp /tmp/angui.json ${WORKSPACE}/angui.json
                  chown jenkins:jenkins ${WORKSPACE}/angui.json
                  chmod 755 ${WORKSPACE}/angui.json"""
            }
        }
        stage('Deploy to ECS') {
            steps {
                
                sh """aws ecs register-task-definition --cli-input-json file://${WORKSPACE}/angui.json
                aws ecs list-task-definitions
                aws ecs run-task --cluster TestCluster --task-definition angui:6 --count 1"""
            }
        }
    }
}
