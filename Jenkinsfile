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

        stage("Check azure") {
            steps {
                script {
                    sh 'az --version'
                }
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                script {
                    sh 'docker pull owasp/dependency-check'
                    sh 'docker run --rm -e USER=$USER -v $PWD:/src -w /src owasp/dependency-check:latest --project devsecops --scan .'
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('SonarQube Server') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Approve based on environment lead') {
            steps {
                script {
                    def approvalChoice = input(
                        id: 'approval',
                        message: 'Please select environment',
                        parameters: [choice(
                            choices: ['PROD', 'STG', 'DEV'],
                            description: 'Select an environment:',
                            name: 'envType'
                        )]
                    )
                    env.approval = approvalChoice
                    env.approverId = env.BUILD_USER
                    echo "Approved by ${env.approverId} (${env.approval})"
                }
            }
        }

        stage("get-credentials") {
            when {
                expression { env.approval == 'PROD' || env.approval == 'STG' || env.approval == 'DEV' }
            }
            steps {
                script {
                    if (env.approval == 'PROD') {
                        echo 'I only execute on the master branch'
                        sh 'kubectl delete deployment'
                        sh 'kubectl apply -f deployment.yml'
                        def deployStatus = sh(script: 'kubectl get deployment', returnStatus: true)
                        if (deployStatus == 0) {
                            echo 'Deployment successful.'
                        } else {
                            error('Deployment failed. Check logs for troubleshooting.')
                        }
                    } else if (env.approval == 'STG' || env.approval == 'DEV') {
                        sh 'kubectl apply -f deployment.yml'
                        def deployStatus = sh(script: 'kubectl get deployment', returnStatus: true)
                        if (deployStatus == 0) {
                            echo 'Deployment successful.'
                        } else {
                            error('Deployment failed. Check logs for troubleshooting.')
                        }
                    }
                }
            }
        }

        stage("confirm deploy") {
            when {
                expression { env.approval == 'PROD' }
            }
            steps {
                script {
                    input(
                        id: 'deploy-confirmation',
                        message: 'Confirmation: Was the deploy successful? (Yes or No)',
                        parameters: [choice(
                            choices: ['Yes', 'No'],
                            description: 'Select an option:',
                            name: 'confirmDeploy'
                        )]
                    )
                    if (env.confirmDeploy == 'Yes') {
                        echo 'Deploy confirmed.'
                    } else {
                        error('Deploy not confirmed. Check logs for troubleshooting.')
                    }
                }
            }
        }

        stage("notify failure") {
            when {
                expression { currentBuild.resultIsWorseOrEqualTo('FAILURE') }
            }
            steps {
                script {
                    error('Pipeline failed. Check logs for troubleshooting.')
                }
            }
        }

        stage("notify success") {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    echo 'Pipeline was successful. Deployment was confirmed.'
                }
            }
        }
    }
}
