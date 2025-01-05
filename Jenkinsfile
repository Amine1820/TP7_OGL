pipeline {
    agent any

     environment {
            // Define environment variables if needed (like Mailtrap details)
            SMTP_SERVER = 'sandbox.smtp.mailtrap.io'
            SMTP_USERNAME = 'a0c4d400e70cc1'
            SMTP_PASSWORD = 'fa3dfbec2662df'
            TO_EMAIL = 'la_melzi@esi.dz'
        }


    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                bat './gradlew test'
            }
            post {
                always {
                    echo 'Archiving test results...'
                    junit 'build/test-results/**/*.xml'
                }
            }
        }

        stage('Code Coverage') {
            steps {
                echo 'Generating Jacoco code coverage report...'
                bat './gradlew jacocoTestReport'
            }
            post {
                always {
                    echo 'Publishing Jacoco report...'
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'build/reports/jacoco/test/html',
                        reportFiles: 'index.html',
                        reportName: 'Jacoco Code Coverage Report'
                    ])
                }
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('sonar') { // Ensure SonarQube is configured in Jenkins
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Code Quality') {
            steps {
                echo 'Checking Quality Gates...'
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                bat './gradlew clean build -x test'
            }
        }

        stage('Publish to Maven') {
            steps {
                echo 'Publishing artifacts to Maven repository...'
                bat "./gradlew publish"
            }
        }
    }


}
