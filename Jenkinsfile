pipeline {
    agent any

    stages {
        stage("Check docker") {
            steps {
                script {
                    sh 'docker --version'
                    sh 'docker compose --version'
                }
            }
        }

        stage('Checkout do código') {
            steps {
                git(url: 'https://github.com/Leonardo-009/conversao-temperatura.git', branch: 'main')
            }
        }

        stage('Construção da imagem Docker') {
            steps {
                script {
                    // Build the Docker image
                    def dockerapp = docker.build("leonardo/conversao-temperatura:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Docker push') {
            steps {
                script {
                    // Push the Docker image to the registry
                    dockerapp.push('latest')
                    dockerapp.push("${env.BUILD_ID}")
                }
            }
        }
    }
}
