pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh './gradlew test'
                junit '**/build/test-results/**/*.xml'  // Archivage des résultats
                cucumberReports jsonPath: '**/build/reports/cucumber/*.json'
            }
        }
        stage('Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh './gradlew sonarqube'
                }
            }
        }
        stage('Code Quality') {
            steps {
                echo 'Checking Quality Gates...'
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Building application...'
                sh './gradlew build jar'
                echo 'Generating documentation...'
                sh './gradlew javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying JAR to Maven repository...'
                withCredentials([usernamePassword(credentialsId: 'mymavenrepo-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh './gradlew publish -PmavenUser=$USER -PmavenPassword=$PASS'
                }
            }
        }
        stage('Notification') {
            steps {
                echo 'Sending success notification...'
                mail bcc: '', body: 'Le pipeline s\'est terminé avec succès.', subject: 'Pipeline Jenkins: Succès', to: 'team@example.com'
                slackSend(channel: '#development', message: 'Déploiement réussi du projet.')
            }
        }
    }
    post {
        failure {
            echo 'Pipeline failed. Sending failure notification...'
            mail bcc: '', body: 'Une erreur est survenue dans le pipeline Jenkins.', subject: 'Pipeline Jenkins: Échec', to: 'team@example.com'
            slackSend(channel: '#development', message: 'Échec du pipeline Jenkins.')
        }
    }
}
