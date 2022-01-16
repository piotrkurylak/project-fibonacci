pipeline {
    agent {label 'linux'}
    options{
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
    }
    stages{
        stage('Build react image for tests'){
            options{
                timeout(time: 5, unit: "MINUTES")
            }
            steps{
                sh 'docker build -t pkur1/project-fibonacci-react-tests:latest -f ./client/Dockerfile.dev ./client'
            }
        }
        stage('Run react tests'){
            options{
                timeout(time: 2, unit: "MINUTES")
            }
            steps{
                sh 'docker run -e CI=true pkur1/project-fibonacci-react-tests:latest npm run test'
        }
        }
        stage('Build prod images'){
            options{
                timeout(time: 5, unit: "MINUTES")
            }
            steps{
                sh '''
                    docker build -t pkur1/project-fibonacci-client:latest ./client
                    docker build -t pkur1/project-fibonacci-api:latest ./server
                    docker build -t pkur1/project-fibonacci-worker:latest ./worker
                    docker build -t pkur1/project-fibonacci-nginx:latest ./nginx
                '''
            }
        }
        stage('Login to Docker Hub'){
            environment{
                DOCKER_HUB_ACCESS_KEY = credentials('docker-hub-pkur1-access-key')
            }
            options{
                timeout(time: 2, unit: "MINUTES")
            }
            steps{
                sh 'echo ${DOCKER_HUB_ACCESS_KEY} | docker login -u pkur1 --password-stdin'
            }
        }
        stage('Push images to Docker Hub'){
            options{
                timeout(time: 5, unit: "MINUTES")
            }
            steps{
                sh '''
                    docker push pkur1/project-fibonacci-client:latest
                    docker push pkur1/project-fibonacci-api:latest
                    docker push pkur1/project-fibonacci-worker:latest
                    docker push pkur1/project-fibonacci-nginx:latest
                '''
            }
        }
        stage('Deploy to production'){
            environment{
                EB_APP_VERSION = "latest"
                EB_APP_ENVIRONMENT_NAME = "project-fibonacci-env"
                EB_APP_NAME = "project-fibonacci-app"
                AWS_ACCESS_KEY_ID = credentials('jenkins-aws-access-key-id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-access-key')
                AWS_S3_BUCKET_NAME = 'project-fibonacci-app'
                ARTIFACT_NAME = 'docker-compose.yml'
                AWS_DEFAULT_REGION = 'eu-west-1'
            }
            options{
                timeout(time: 10, unit: "MINUTES")
            }
            steps{
                sh '''
                    aws s3 cp /home/admin/jenkins/workspace/Project_Fibonacci_dev/${ARTIFACT_NAME} s3://${AWS_S3_BUCKET_NAME}/app/${ARTIFACT_NAME}
                    aws elasticbeanstalk update-environment --application-name ${EB_APP_NAME} --environment-name ${EB_APP_ENVIRONMENT_NAME} --version-label project-fibonacci-version-1
                '''
            }
        }
    }
    post{
        always{
        sh 'docker logout'
        }
    }
}