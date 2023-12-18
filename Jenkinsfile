#!/usr/bin/env groovy
pipeline {
    agent any

    environment {
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

        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh "sudo docker build -t $DOCKER_IMAGE_NAME ."
                    sh "sudo docker tag nodejs:latest 915270456781.dkr.ecr.ap-southeast-2.amazonaws.com/nodejs:latest"
                    sh "sudo aws configure --accessKeyVariable=$AWS_ACCESS_KEY_ID --secretKeyVariable=$AWS_SECRET_ACCESS_KEY"
                    sh "sudo docker push 915270456781.dkr.ecr.ap-southeast-2.amazonaws.com/nodejs:latest"
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
