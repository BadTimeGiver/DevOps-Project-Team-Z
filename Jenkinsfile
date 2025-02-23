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
                    // Appliquer la nouvelle configuration (qui inclut le liveness probe)
                    sh 'kubectl apply -f Deployment.yaml'

                    // Attendre que le déploiement soit prêt
                    sh 'kubectl rollout status deployment/m2-devops-project --timeout=60s'

                    // Lancer le port-forwarding en arrière-plan
                    sh 'kubectl port-forward service/m2-devops-project-service 8081:8081 &'
                }
            }
        }

        stage("Test endpoint") {
            steps {
                sh 'curl --fail http://localhost:8081/whoami || exit 1'
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
                sh 'echo "Create Prod Environment"'
            }
        }
    }
}
