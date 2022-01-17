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
                sh 'docker build -t pkur1/project-fibonacci-react-tests:${BUILD_NUMBER} -f ./client/Dockerfile.dev ./client'
            }
        }
        stage('Run react tests'){
            options{
                timeout(time: 2, unit: "MINUTES")
            }
            steps{
                sh 'docker run -e CI=true pkur1/project-fibonacci-react-tests:${BUILD_NUMBER} npm run test'
        }
        }
        stage('Build prod images'){
            options{
                timeout(time: 5, unit: "MINUTES")
            }
            steps{
                sh '''
                    docker build -t pkur1/project-fibonacci-client:${BUILD_NUMBER} ./client
                    docker build -t pkur1/project-fibonacci-api:${BUILD_NUMBER} ./server
                    docker build -t pkur1/project-fibonacci-worker:${BUILD_NUMBER} ./worker
                    docker build -t pkur1/project-fibonacci-nginx:${BUILD_NUMBER} ./nginx
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
                    docker push pkur1/project-fibonacci-client:${BUILD_NUMBER}
                    docker push pkur1/project-fibonacci-api:${BUILD_NUMBER}
                    docker push pkur1/project-fibonacci-worker:${BUILD_NUMBER}
                    docker push pkur1/project-fibonacci-nginx:${BUILD_NUMBER}
                '''
            }
        }
        stage('Deploy to production'){
            environment{
                EB_APP_ENVIRONMENT_NAME = "project-fibonacci-env"
                EB_APP_NAME = "project-fibonacci-app"
                AWS_ACCESS_KEY_ID = credentials('jenkins-aws-access-key-id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-access-key')
                AWS_S3_BUCKET_NAME = 'project-fibonacci-app'
                ENTRYPOINT = 'docker-compose.yml'
                AWS_DEFAULT_REGION = 'eu-west-1'
            }
            options{
                timeout(time: 10, unit: "MINUTES")
            }
            steps{
                sh '''
                    aws s3 cp /home/admin/jenkins/workspace/Project_Fibonacci_dev/${ENTRYPOINT} s3://${AWS_S3_BUCKET_NAME}/${ENTRYPOINT}
                    aws elasticbeanstalk create-application-version --application-name ${EB_APP_NAME} --version-label $BUILD_NUMBER --source-bundle S3Bucket=$AWS_S3_BUCKET_NAME,S3Key=$ENTRYPOINT
                    aws elasticbeanstalk update-environment --application-name ${EB_APP_NAME} --environment-name ${EB_APP_ENVIRONMENT_NAME} --version-label $BUILD_NUMBER
                '''
            }
        }
    }
    post{
        environment{
            MAIL = credentials('jenkins-notifications')
        }
        always{
            sh 'docker logout'
        }
        failure {
            mail to: '${MAIL}',
                subject: "Failure on build: ${env.BUILD_NUMBER}",
                body: "I am sorry, something went wrong, check ${env.BUILD_URL}"
    }
    }
}