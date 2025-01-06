pipeline {
    agent any

    // Since we're not using any environment variables, we can remove the environment block
    // If you need to add environment variables later, you can uncomment and add them
    /*
    environment {
        // Add environment variables here if needed
        // Example:
        // MAKE_PATH = tool name: 'make'
    }
    */

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
                // Make sure 'make' is installed and available in the PATH
                sh "make check"
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
    }

    // Optional: Add post actions for success/failure handling
    post {
        failure {
            echo 'The Pipeline failed :('
        }
        success {
            echo 'The Pipeline completed successfully :)'
        }
    }
}