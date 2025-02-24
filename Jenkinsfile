pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BadTimeGiver/DevOps-Project-Team-Z'
            }
        }

        stage("Launch Kubernetes") {
            steps {
                sh """
                    minikube delete
                    minikube start
                """
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                      usernameVariable: 'DOCKERHUB_USERNAME',
                                                      passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                        sh 'docker build -t serquand/project-devops:latest .'
                        sh 'docker push serquand/project-devops:latest'
                    }
                }
            }
        }

        stage("Create Dev Environment") {
            steps {
                script {
                    sh """
                        kubectl create namespace development --dry-run=client -o yaml | kubectl apply -f -
                        kubectl apply -f Deployment.yaml -n development
                        kubectl rollout status deployment/m2-devops-project -n development --timeout=60s
                        kubectl port-forward service/m2-devops-project-service -n development 8081:8081 &
                    """
                }
            }
        }

        stage("Test endpoint") {
            steps {
                sh 'curl --fail http://localhost:8084/whoami || exit 1'
            }
            post {
                failure {
                    echo "Something went wrong when trying to connect the API! The new version won't be released!"
                }
                success {
                    echo "The new version will be released!"
                }
            }
        }

        stage("Create Prod Environment") {
            steps {
                script {
                    sh """
                        kubectl create namespace production --dry-run=client -o yaml | kubectl apply -f -
                        kubectl apply -f Deployment.yaml -n production
                        kubectl rollout status deployment/m2-devops-project -n production --timeout=60s
                        kubectl port-forward service/m2-devops-project-service -n production 8082:8081 &
                    """
                }
            }
        }
    }
}
