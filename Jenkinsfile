pipeline {
    agent any

    tools {
        nodejs 'node25'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'feature', url: 'https://github.com/sunnymokkala/Book-My-Show.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('bookmyshow-app') {
                    sh 'npm install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('bookmyshow-app') {
                    withSonarQubeEnv('sonarqube') {
                        sh """
                        ${tool 'sonar-scanner'}/bin/sonar-scanner \
                        -Dsonar.projectKey=bookmyshow \
                        -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('bookmyshow-app') {
                    sh 'docker build -t bms-app .'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                docker rm -f bookmyshow-container || true
                docker run -d -p 3000:3000 --name bookmyshow-container bms-app
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "Jenkins Build SUCCESS",
                body: """
Pipeline executed successfully.

Project: BookMyShow
Build Number: ${env.BUILD_NUMBER}
Job: ${env.JOB_NAME}

Application deployed successfully.
""",
                to: "sunnymokkala@gmail.com"
            )
        }

        failure {
            emailext(
                subject: "Jenkins Build FAILED",
                body: """
Pipeline execution FAILED.

Project: BookMyShow
Build Number: ${env.BUILD_NUMBER}
Job: ${env.JOB_NAME}

Check Jenkins console logs.
""",
                to: "sunnymokkala@gmail.com"
            )
        }

        always {
            echo "Pipeline finished."
        }
    }
}
