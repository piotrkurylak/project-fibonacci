pipeline {
    agent {label 'linux'}
    environment{
        DOCKER_HUB_ACCESS_KEY = credentials('docker-hub-pkur1-access-key')
        EB_APP_VERSION = "${BUILD_NUMBER}"
        EB_APP_ENVIRONMENT_NAME = "project-fibonacci-env"
        EB_APP_NAME = "project-fibonacci-app"
        AWS_ACCESS_KEY_ID = credentials('jenkins-aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-access-key')
        AWS_S3_BUCKET_NAME = 'project-fibonacci-app'
        ARTIFACT_NAME = 'docker-compose.yml'
    }
    options{
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
    }
    stages{
        stage('Build react image for tests'){
            steps{
                sh 'docker build -t pkur1/project-fibonacci-react-tests:${BUILD_NUMBER} -f ./client/Dockerfile.dev ./client'
            }
        }
        stage('Run react tests'){
            steps{
                sh 'docker run -e CI=true pkur1/project-fibonacci-react-tests:${BUILD_NUMBER} npm run test'
        }
        }
        stage('Build prod images'){
            steps{
                sh '''
                    docker build -t pkur1/project-fibonacci-client:${BUILD_NUMBER} ./client
                    docker build -t pkur1/project-fibonacci-server:${BUILD_NUMBER} ./server
                    docker build -t pkur1/project-fibonacci-worker:${BUILD_NUMBER} ./worker
                    docker build -t pkur1/project-fibonacci-nginx:${BUILD_NUMBER} ./nginx
                '''
            }
        }
        stage('Login to Docker Hub'){
            steps{
                sh 'echo ${DOCKER_HUB_ACCESS_KEY} | docker login -u pkur1 --password-stdin'
            }
        }
        stage('Push images to Docker Hub'){
            steps{
                sh '''
                    docker push pkur1/project-fibonacci-client:${BUILD_NUMBER}
                    docker push pkur1/project-fibonacci-server:${BUILD_NUMBER}
                    docker push pkur1/project-fibonacci-worker:${BUILD_NUMBER}
                    docker push pkur1/project-fibonacci-nginx:${BUILD_NUMBER}
                '''
            }
        }
        stage('Deploy to production'){
            steps{
                sh '''
                    aws s3 cp /home/admin/jenkins/workspace/Project_Fibonacci_dev/${ARTIFACT_NAME} s3://${AWS_S3_BUCKET_NAME}/${ARTIFACT_NAME}
                    aws elasticbeanstalk update-environment --application-name ${EB_APP_NAME} --environment-name ${EB_APP_ENVIRONMENT_NAME} --version-label ${AWS_EB_APP_VERSION}
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