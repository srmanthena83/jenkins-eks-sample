pipeline {
    agent any

    tools {
        nodejs 'NodeJS 18' // Name as configured in Global Tool Configuration
    }    

    environment {
        // Define environment variables
        AWS_REGION = 'us-east-1'
        ECR_REPO = '953816747017.dkr.ecr.us-east-1.amazonaws.com/jenkins-eks-sample'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('kube-config') // If using Kubernetes credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/srmanthena83/jenkins-eks-sample.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run test'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: 'aws-credentials') {
                    sh '$(aws ecr get-login --no-include-email --region $AWS_REGION)'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    dockerImage.push()
                }
            }
        }

        // stage('Deploy to EKS') {
        //     steps {
        //         withAWS(region: "${AWS_REGION}", credentials: 'aws-credentials') {
        //             sh '''
        //                 kubectl set image deployment/jenkins-eks-sample-deployment jenkins-eks-sample=${ECR_REPO}:${IMAGE_TAG} --record
        //                 kubectl rollout status deployment/jenkins-eks-sample-deployment
        //             '''
        //         }
        //     }
        // }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

