pipeline {
    agent any

    stages {
        stage('Test Docker') {
            steps {
                script {
                    sh 'export PATH=$PATH:/opt/homebrew/bin && docker --version'
                }
            }
        }


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
    }
}
