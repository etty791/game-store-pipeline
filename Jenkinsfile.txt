pipeline {
    agent any

    environment {
        APP_NAME = "GameStore"
    }

    options {
        timeout(time: 15, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Downloading ${APP_NAME} source code from GitHub..."
                checkout scm
            }
        }

        stage('Build Game Assets') {
            steps {
                echo "Compiling game images and catalogs..."
                bat 'echo building assets...'
                stash includes: 'assets/**', name: 'game-assets', allowEmpty: true
            }
        }

        stage('Quality Assurance') {
            parallel {
                stage('Inventory Tests') {
                    steps {
                        echo "Checking if all games are in stock..."
                }
}
                stage('Payment Gateway Tests') {
                    steps {
                        echo "Testing credit card integration..."
                    }
                }
            }
        }

        stage('Manual Approval for Sale') {
            steps {
                script {
                    input message: "האם את בטוחה שאת רוצה להעלות את חנות המשחקים לאוויר?", ok: "כן, שחרר את המשחקים!"
                }
            }
        }

        stage('Deploy to Store') {
            when { branch 'main' } 
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'store-admin-creds', usernameVariable: 'ADMIN_USER', passwordVariable: 'ADMIN_PASS')
                ]) {
                    echo "Deploying to production as ${ADMIN_USER}..."
                    unstash 'game-assets'
                    bat 'echo Deployment complete!'
                }
            }
        }
    }

    post {
        success {
            echo "The Store is UP! All tests passed."
        }
        failure {
            echo "Deployment failed! Check the logs to see which game crashed the store."
        }
        always {
            echo "Cleaning workspace for the next gamer..."
            cleanWs()
        }
    }
}

