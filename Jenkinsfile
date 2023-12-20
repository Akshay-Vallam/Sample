#!/usr/bin/env groovy
pipeline {
    agent any

    environment {
        ECR_REPO_URL = '915270456781.dkr.ecr.ap-southeast-2.amazonaws.com/nodejs'
        IMAGE_TAG = 'latest' // Change this to your desired tag
        AWS_REGION = 'ap-southeast-2'
        ECS_CLUSTER = 'Sample'
        ECS_SERVICE = 'Sample'
        DOCKER_IMAGE_NAME = 'nodejs'
        AWS_CREDENTIALS = 'AWS_Credentials' // Should be configured in Jenkins Credentials
    }

    stages {
       stage('Git Checkout') {
            steps {
                script {
                    git branch: 'main',
                        url: 'https://github.com/Akshay-Vallam/NodeJS-Demo'
                }
            }
        }
        stage('ECR Login and Push') {
            steps {
                script {
                    // Login to AWS ECR
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        credentialsId: 'AWS_Credentials',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        sh "sudo aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}"
                    }

                    // Build and push Docker image to ECR
                    sh "sudo docker build -t ${ECR_REPO_URL}:${IMAGE_TAG} ."
                    sh "sudo docker push ${ECR_REPO_URL}:${IMAGE_TAG}"
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                script {
                    // Configure AWS credentials
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: "$AWS_CREDENTIALS"]]) {
                        // Update ECS task definition with the new Docker image
                        sh "aws ecs register-task-definition --cli-input-json file=ecs-task-definition.json"

                        // Update ECS service to use the new task definition
                        sh "aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition $DOCKER_IMAGE_NAME:latest"
                    }
                }
            }
        }
    }
}
