pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'mon-credential-id', url: 'https://github.com/BadTimeGiver/DevOps-Project-Team-Z'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t project:latest .'
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
                    echo "Create Dev Environment"
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
