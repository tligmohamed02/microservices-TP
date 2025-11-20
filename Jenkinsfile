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
        stage('Deploy to All Clusters') {
                    parallel {
                        stage('Deploy to Minikube') {
                            steps {
                                dir('microservices-TP') {
                                    script {
                                        kubeconfig(credentialsId: 'wslKubernetesConfigFile', serverUrl: '') {
                                            echo "--- DEPLOYING TO MINIKUBE ---"
                                            sh 'kubectl --context minikube apply -f service.yaml'
                                            sh 'kubectl --context minikube apply -f deployment.yaml'
                                            sh "kubectl --context minikube set image deployment/my-country-service country-service-container=mohamed357/my-country-service:${env.BUILD_NUMBER}"
                                            sh 'kubectl --context minikube rollout status deployment/my-country-service'
                                        }
                                    }
                                }
                            }
                        }

                        stage('Deploy to Docker Desktop') {
                            steps {
                                dir('microservices-TP') {
                                    script {
                                        kubeconfig(credentialsId: 'wslKubernetesConfigFile', serverUrl: '') {
                                            echo "--- DEPLOYING TO DOCKER DESKTOP ---"
                                            sh 'kubectl --context docker-desktop apply -f service.yaml'
                                            sh 'kubectl --context docker-desktop apply -f deployment.yaml'
                                            sh "kubectl --context docker-desktop set image deployment/my-country-service country-service-container=mohamed357/my-country-service:${env.BUILD_NUMBER}"
                                            sh 'kubectl --context docker-desktop rollout status deployment/my-country-service'
                                        }
                                    }
                                }
                            }
                        }
                    }
                }

//          stage('Deploy to Kubernetes') {
//                     steps {
//                         dir('microservices-TP') {
//                             script {
//                                 kubeconfig(credentialsId: 'kubeconfig-file', serverUrl: '') {
//                                     echo "--- Applying base manifests ---"
//                                     // 1. Applique les fichiers YAML pour s'assurer que les objets Service et Deployment existent ou sont à jour.
//                                     sh 'kubectl apply -f service.yaml'
//                                     sh 'kubectl apply -f deployment.yaml'
//
//                                     echo "--- Setting deployment image to version ${env.BUILD_NUMBER} ---"
//                                     // 2. Met à jour l'image du déploiement pour déclencher un "rolling update" avec la nouvelle version.
//                                     sh "kubectl set image deployment/my-country-service country-service-container=mohamed357/my-country-service:${env.BUILD_NUMBER}"
//
//                                     //echo "--- Verifying deployment rollout ---"
//                                     // (Optionnel mais recommandé) Attendre que le déploiement soit terminé avec succès.
//                                     //sh 'kubectl rollout status deployment/my-country-service'
//                                 }
//                             }
//                         }
//                     }
//                 }

    }
}

