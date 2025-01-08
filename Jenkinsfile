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
                slackSend message: "Build and tests passed successfully! "
            }
            failure {
                slackSend message: "Build or tests failed! Check Jenkins for details."
            }
            unstable {
                slackSend message: "Build or tests are unstable. Review the logs."
            }
        }
}
