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
                    slackSend message: "Build and tests passed successfully!"
                    mail(
                        to: 'la_melzi@esi.dz',
                        subject: 'Jenkins Build Success',
                        body: 'The build and tests were successful. All tests passed.'
                    )
                }
                failure {
                    slackSend message: "Build or tests failed! Check Jenkins for details."
                    mail(
                        to: 'la_melzi@esi.dz',
                        subject: 'Jenkins Build Failure',
                        body: 'The build or tests failed. Please check the Jenkins console output for more details.'
                    )
                }
                unstable {
                    slackSend message: "Build or tests are unstable. Review the logs."
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
