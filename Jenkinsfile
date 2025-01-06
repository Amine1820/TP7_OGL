pipeline {
    agent any

    environment {

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
                sh " make check"
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }


    }
}