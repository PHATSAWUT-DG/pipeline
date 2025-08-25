pipeline {
    agent any
    parameters {
        booleanParam(name: 'RUN_DEPLOY', defaultValue: true, description: 'Should we deploy?')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Select environment')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
                // Simulate build
                sh 'sleep 2'
            }
        }
        stage('Test in Parallel') {
            parallel {
                stage('Unit Tests') {
                    when {
                        succeeded()
                        
                    }
                    steps {
                        echo 'Running unit tests...'
                        sh 'echo \"All tests passed!\" > results.txt'
                        archiveArtifacts artifacts: 'results.txt', fingerprint: true
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests...'
                        sh 'sleep 2'
                    }
                }
                stage('Linux Test') {
                    steps {
                        echo 'Testing on Linux...'
                    }
                }
                stage('Windows Test') {
                    steps {
                        echo 'Testing on Windows...'
                    }
                }
            }
        }
        stage('Approval') {
            when {
                expression { return params.RUN_DEPLOY }
            }
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    input "Do you want to proceed with deployment?"
                }
            }
        }
        stage('Deploy') {
            when {
                allOf {
                    expression { return params.RUN_DEPLOY }
                }
            }
            steps {
                echo "Deploying to environment: ${params.ENV}"
                // Simulate deploy
                sh 'sleep 2'
            }
        }
    }
    post {
        success {
            echo '✅ Pipeline finished successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs!'
        }
        always {
            echo 'Pipeline completed (success or failure).'
        }
    }
}