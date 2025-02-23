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
                    minikube docker-env
                    minikube ssh -- docker images
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        eval $(minikube -p minikube docker-env)
                        docker build -t project:latest .
                    """
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    // Stop & remove old container
                    sh 'docker ps -q --filter "name=project" | xargs -r docker stop'
                    sh 'docker ps -a -q --filter "name=project" | xargs -r docker rm'

                    // Run new container
                    sh 'docker run -d --name project_container -p 8081:8081 project:latest'
                }
            }
        }


        stage("Create Dev Environment") {
            steps {
                sh """
                    # Load the image
                    minikube image load project:latest

                    # Create the Deployment & Pods
                    kubectl apply -f Deployment.yaml

                    # Launch kubectl
                    kubectl port-forward service/m2-devops-project-service 8081:8081
                """
            }
        }

        stage("Test endpoint") {
            steps {
                sh """
                    curl --fail http://localhost:8082/whoami || exit 1
                """
            }
            post {
                failure {
                    echo "Something went wrong when trying to connect the API ! The new version won't be released !"
                }
                success {
                    echo "The new version will be released !"
                }
            }
        }

        stage("Create Prod Environment") {
            steps {
                sh """
                    echo "Create Prod Environment"
                """
            }
        }
    }
}
