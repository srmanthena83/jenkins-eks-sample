pipeline {
    agent {
        docker {
            image 'node:18' // Official Node.js Docker image
            // args '-v /var/run/docker.sock:/var/run/docker.sock' // If Docker commands are needed
            args '-u root' // Run as root to ensure permissions (if necessary)
        }
    }  

    environment {
        // Define environment variables
        AWS_REGION = 'us-east-1'
        ECR_REPO = '953816747017.dkr.ecr.us-east-1.amazonaws.com/jenkins-eks-sample'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBE_CONFIG = credentials('kube-config') // If using Kubernetes credentials
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm" // Custom npm cache directory
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/srmanthena83/jenkins-eks-sample.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mkdir -p $NPM_CONFIG_CACHE'    // Ensure the cache directory exists
                sh 'npm install'
                sh 'npm run test'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = sh(script: '/bin/docker build -t ${ECR_REPO}:${IMAGE_TAG} .', returnStdout: true)
                    //dockerImage = docker.build("${ECR_REPO}:${IMAGE_TAG}")
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

