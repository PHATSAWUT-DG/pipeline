pipeline {
    agent any
    stages {
        stage("Build") {
            steps {
                echo "Build stage."
            }
        }
        stage("Test") {
            steps {
                echo "Test stage."
            }
        }
        stage("Release") {
            steps {
                echo "Release stage."
            }
        }
         post {
            success {
                echo 'Pipeline completed successfully 🎉'
            }
            failure {
                echo 'Pipeline failed ❌'
            }
}

    }
}