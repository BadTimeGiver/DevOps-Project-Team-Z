pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/BadTimeGiver/DevOps-Project-Team-Z'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t project .'
                }
            }
        }
        stage('Deploy to Docker') {
            steps {
                script {
                    sh 'docker run -d -p 8081:8081 project'
                }
            }
        }
    }
}