// Pipeline to build image and push to ECR on AWS.
// Write by DevOps and Cloud engineer Franklin Foko fokofranklin47@gmail.com
// All rigths reserved by FELICITY COMPANY LTD 2023

pipeline {
    agent any
    options {
        //skipDefaultCheckout()
        skipStagesAfterUnstable()
    }

    environment {
        AWS_ACCOUNT_ID="989481297723"
        AWS_DEFAULT_REGION="us-east-1"  
        IMAGE_REPO_NAME="kafka-zookeeper"
        IMAGE_TAG="latest"
        BRANCH_NAME = "${GIT_BRANCH.split("/")[1]}"
        registryCredential = "aws-credentials"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages {
        
         stage('Clone repository') {
            steps {
                script{
                    sh 'printenv'
                    echo 'Pulling...' + env.BRANCH_NAME
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                script{
                    slackSend(message: "${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nBuild commencé", channel: "zunihealth")
                    if (env.BRANCH_NAME == 'develop') {
                        echo env.BRANCH_NAME
                        app = docker.build("${REPOSITORY_URI}:develop")
                    }
                    if (env.BRANCH_NAME == 'main') {
                        echo env.BRANCH_NAME
                        app = docker.build("${REPOSITORY_URI}:latest")
                    } else {
                        echo env.BRANCH_NAME
                        app = docker.build("${REPOSITORY_URI}")
                    }
                    slackSend(message: "${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nBuild success", channel: "zunihealth")
                }
            }
        }
        stage('Push Image on ECR') {
            steps {
                script{
                    slackSend(message: "${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nPush docker image on ECR start", channel: "zunihealth")
                    if (env.BRANCH_NAME == 'develop') {
                        docker.withRegistry("https://${REPOSITORY_URI}", "ecr:${AWS_DEFAULT_REGION}:"+ registryCredential) {
                            echo env.BRANCH_NAME
                            app.push(env.BRANCH_NAME)
                        }
                    }
                    if (env.BRANCH_NAME == 'main') {
                        docker.withRegistry("https://${REPOSITORY_URI}", "ecr:${AWS_DEFAULT_REGION}:"+ registryCredential) {
                            echo env.BRANCH_NAME
                            app.push("latest")
                        } 
                    } else {
                        echo env.BRANCH_NAME
                        
                    }
                    slackSend(message: "${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nPush image success", channel: "zunihealth")
                }
            }
        }
    

        stage('Deploy on AWS') {
            steps {
                script{
                    slackSend(message: "${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nDeploy on AWS start", channel: "zunihealth")
                    if (env.BRANCH_NAME == 'main') {
                        echo env.BRANCH_NAME
                    //    sh 'aws ecs update-service --force-new-deployment --service kafka-zookeeper-Service --task-definition kafka-zookeeper-TaskDefinition:1 --cluster kafka-zookeeper-Cluster  --desired-count 1  --profile server'
                    } 
                    if (env.BRANCH_NAME == 'develop') {
                        echo env.BRANCH_NAME
                        sh 'docker build -t kafka-zookeeper .'
                        sh 'docker run -p 2181:2181 -p 9092:9092 -e ADVERTISED_HOST=localhost kafka-zookeeper'

                    }
                    slackSend(message: "${env.JOB_NAME} #${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nDeploy on AWS success", channel: "zunihealth")
                }
            }
        }
    }
    
    post {
        success {
            echo " success"
              slackSend(color: "good", message: "${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nPipeline CI/CD effectué avec succès.", channel: "zunihealth")
        }
        failure {
            echo "failed"
              slackSend(failOnError:true, color: "danger", message:"${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)\nÉchec de pipeline CI/CD", channel: "zunihealth")
        }
        
    }
}

