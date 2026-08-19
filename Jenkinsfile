pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo '========================================'
                echo 'GitHub -> Jenkins webhook received'
                echo '========================================'

                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo '========================================'
                echo 'Jenkins pipeline is working successfully'
                echo '========================================'

                bat 'echo Current workspace: %CD%'
            }
        }
    }

    post {
        success {
            echo 'BUILD SUCCESSFUL'
        }

        failure {
            echo 'BUILD FAILED'
        }
    }
}