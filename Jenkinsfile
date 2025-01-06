pipeline {
    agent any

    environment {
        // Load credentials from Jenkins credentials store
        SMTP_CREDS = credentials('mailtrap-smtp-credentials') // ID of username/password credential
        TO_EMAIL = credentials('notification-email') // ID of secret text credential

        // Load from Jenkins configuration
        SONAR_HOME = tool name: 'SonarScanner' // SonarQube Scanner installation
        JAVA_HOME = tool name: 'JDK11'
        MAVEN_HOME = tool name: 'Maven3'

        // If you have a Jenkins configuration file, you can load properties
        DEPLOY_ENV = "${params.DEPLOY_ENV ?: 'development'}"
    }

    tools {
        maven 'Maven3'
        jdk 'JDK11'
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
                sh "mvn test"
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Code Coverage') {
            steps {
                echo 'Generating JaCoCo code coverage report...'
                sh "mvn verify org.jacoco:jacoco-maven-plugin:prepare-agent"
            }
            post {
                always {
                    jacoco(
                        execPattern: 'target/*.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java',
                        exclusionPattern: 'src/test/*'
                    )
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JaCoCo Code Coverage Report'
                    ])
                }
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('sonar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Publish to Maven') {
            steps {
                echo 'Publishing artifacts to Maven repository...'
                withCredentials([usernamePassword(
                    credentialsId: 'maven-repo-credentials',
                    usernameVariable: 'MAVEN_USERNAME',
                    passwordVariable: 'MAVEN_PASSWORD'
                )]) {
                    sh """
                        mvn deploy -DskipTests \
                        -Drepository.username=${MAVEN_USERNAME} \
                        -Drepository.password=${MAVEN_PASSWORD}
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                emailext (
                    to: "${TO_EMAIL}",
                    subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Success",
                    body: """
                        <p>Build Status: Success</p>
                        <p>Job: ${env.JOB_NAME}</p>
                        <p>Build Number: ${env.BUILD_NUMBER}</p>
                        <p>Build URL: ${env.BUILD_URL}</p>
                    """,
                    mimeType: 'text/html',
                    attachLog: true
                )
            }
        }

        failure {
            script {
                emailext (
                    to: "${TO_EMAIL}",
                    subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - Failed",
                    body: """
                        <p>Build Status: Failed</p>
                        <p>Job: ${env.JOB_NAME}</p>
                        <p>Build Number: ${env.BUILD_NUMBER}</p>
                        <p>Build URL: ${env.BUILD_URL}</p>
                    """,
                    mimeType: 'text/html',
                    attachLog: true
                )
            }
        }
    }
}