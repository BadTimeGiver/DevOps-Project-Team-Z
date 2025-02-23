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

        }

        stage("Test endpoint") {
            steps {
                sh """
                    curl --fail http://localhost:8082/whoami || exit 1
                """
            }
        }

        stage("Create Prod Environment") {
            steps {
                
            }
        }
    }
}
