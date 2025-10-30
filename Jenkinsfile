pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'master', url: 'https://github.com/tligmohamed02/microservices-TP.git'
            }
        }

        stage('Build maven' ) {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build DockerFile') {
            steps {
                sh 'docker build . -t my-country-service:$BUILD_NUMBER'
                withCredentials([string(credentialsId: 'dockerhub_pwd', variable: 'DOCKERHUB_PASSWORD')]) {
                    sh 'docker login -u mohamed357 -p ${DOCKERHUB_PASSWORD}'
                }
                sh 'docker tag my-country-service:$BUILD_NUMBER mohamed357/my-country-service:$BUILD_NUMBER'
                sh 'docker push mohamed357/my-country-service:$BUILD_NUMBER'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-file', serverUrl: '') {
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                    }
                }
            }
        }
    }
}
