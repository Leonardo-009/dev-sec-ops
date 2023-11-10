pipeline {
    agent any

    stages {
        stage('Checkout do código') {
            steps {
                git(url: 'https://github.com/Leonardo-009/conversao-temperatura.git', branch: 'main')
            }
        }
    
        stage('Construção da imagem Docker') {
            steps {
                script {
                    // Build the Docker image
                    dockerapp = docker.build("leonardo/conversao-temperatura:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Docker push') {
            steps {
                script {
                    // Push the Docker image to the registry
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }
    }
}
