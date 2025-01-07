pipeline {
    agent any

    environment {
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
        stage('Cucumber Reports') {
            steps {
                echo 'Archiving Cucumber HTML reports...'
                archiveArtifacts artifacts: 'build/reports/cucumber/html/cucumber-html-reports/*', allowEmptyArchive: true
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
                withSonarQubeEnv('sonar') {
                    bat './gradlew sonar'
                }
            }
        }

       stage('Code Quality') {
            steps {
                echo 'Checking Quality Gates...'
                script {
                    timeout(time: 4, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status == 'FAILED') {
                            error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                        }
                    }
                }
            }
        }


         stage('Build') {
                    steps {
                        script {
                            echo 'Building the project...'
                            // Étape 1 : Génération du fichier Jar
                            bat './gradlew clean build -x test'

                            // Étape 2 : Génération de la documentation
                            echo 'Generating documentation...'
                            bat './gradlew javadoc'

                            // Étape 3 : Archivage du fichier Jar et de la documentation
                            echo 'Archiving artifacts...'
                            archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
                        }
                    }
                }

        stage('Publish to Maven') {
            steps {
                echo 'Publishing artifacts to Maven repository... '
                bat "./gradlew publish"
            }
        }
    }

    post {
        success {
            script {
                // Send email notification for success using Mailtrap SMTP
                mail to: "${TO_EMAIL}",
                     subject: "Deployment Success",
                     body: "The deployment was successful."
            }
        }

        failure {
            script {
                // Send email notification for failure using Mailtrap SMTP
                mail to: "${TO_EMAIL}",
                     subject: "Deployment Failed",
                     body: "The deployment failed. Please check the logs for more details."
            }
        }
    }
}
