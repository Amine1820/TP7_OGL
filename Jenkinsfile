pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Running tests...'
                bat './gradlew test'
            }
            post {
                always {
                    echo 'Archiving test results...'
                    junit 'build/test-results/**/*.xml'
                    echo 'Archiving Cucumber HTML reports...'
                    cucumber '**/reports/*.json'
                    archiveArtifacts artifacts: 'build/reports/cucumber/html/cucumber-html-reports/*', allowEmptyArchive: true
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
                withSonarQubeEnv('sonar') {
                    bat './gradlew sonar'
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    echo 'Checking SonarQube Quality Gate status...'
                    // Wait for the Quality Gate status to be available
                    def qualityGate = waitForQualityGate()
                    // If the Quality Gate failed, stop the pipeline
                    if (qualityGate.status == 'FAILED') {
                        error "Quality Gate failed: ${qualityGate.status}"
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

        stage('Deploy') {
            steps {
                echo 'Publishing artifacts to Maven repository... '
                bat "./gradlew publish"
            }
        }

        stage('Notifications') {
            steps {
                echo 'Sending notifications...'
            }
            post {
                success {
                    slackSend (message: "Build and tests passed successfully! " , color: 'good' )
                    mail(
                        to: 'la_melzi@esi.dz',
                        subject: 'Jenkins Build Success',
                        body: 'The build and tests were successful. All tests passed.'
                    )
                }
                failure {
                    slackSend (message: "Build or tests failed! Check Jenkins for details." , color : 'danger')
                    mail(
                        to: 'la_melzi@esi.dz',
                        subject: 'Jenkins Build Failure',
                        body: 'The build or tests failed. Please check the Jenkins console output for more details.'
                    )
                }
                unstable {
                    slackSend ( message: "Build or tests are unstable. Review the logs. " , color : 'warning')
                    mail(
                        to: 'la_melzi@esi.dz',
                        subject: 'Jenkins Build Unstable',
                        body: 'The build or tests are unstable. Please review the logs for more information.'
                    )
                }
            }
        }
    }
}
