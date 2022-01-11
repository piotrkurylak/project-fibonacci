pipeline {
    agent {label 'linux'}
    environment{
        DOCKER_HUB_ACCESS_KEY = credentials('docker-hub-pkur1-access-key')
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
                    docker build -t pkur1/project-fibonacci-client:${BUILD_NUMBER} -f ./client/Dockerfile ./client
                    docker build -t pkur1/project-fibonacci-api:${BUILD_NUMBER} -f ./api/Dockerfile ./api
                    docker build -t pkur1/project-fibonacci-worker:${BUILD_NUMBER} -f ./worker/Dockerfile ./worker
                    docker build -t pkur1/project-fibonacci-nginx:${BUILD_NUMBER} -f ./nginx/Dockerfile ./nginx
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
                    docker push pkur1/project-fibonacci-api:${BUILD_NUMBER} 
                    docker push pkur1/project-fibonacci-worker:${BUILD_NUMBER} 
                    docker push pkur1/project-fibonacci-nginx:${BUILD_NUMBER} 
                '''
            }
        }
    }
    post{
        sh 'docker logout'
    }
}