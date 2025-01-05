pipeline {
    agent any

    environment {
        MAVEN_REPO_URL = 'https://mymavenrepo.com/repo/RcJxiPx2FgIXYY8orX58/'
        MAVEN_REPO_USER = 'myMavenRepo'
        MAVEN_REPO_PASS = 'Amine'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                bat './gradlew clean build -x test'
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

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('sonar') { // Ensure SonarQube is configured in Jenkins
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Publish to Maven') {
            steps {
                echo 'Publishing artifacts to Maven repository...'
                withCredentials([usernamePassword(
                    credentialsId: 'maven-repo-credentials',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    bat "./gradlew publish -PmavenUser=$USERNAME -PmavenPassword=$PASSWORD"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }

        success {
            echo 'Build completed successfully!'
        }

        failure {
            echo 'Build failed. Please check the logs.'
        }
    }
}
