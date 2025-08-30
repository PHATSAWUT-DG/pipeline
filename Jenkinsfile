pipeline {
    agent any
    
    environment {
        SONAR_TOKEN = 'sqp_4baffa57a1e9bd4f1ae289ac790fb6ca4fdbd487'
        SONAR_HOST_URL = 'http://172.17.0.3:9000'
        PROJECT_KEY = 'test'
        PROJECT_NAME = 'test'
    }

    stages {
        stage('Maven Check') {
            steps {
                script {
                    try {
                        sh 'docker --version'
                        sh 'docker info'
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'find . -name "pom.xml" -type f'
                    } catch (Exception e) {
                        echo "Docker access issue: ${e.getMessage()}"
                        error "Docker is not accessible. Please check Docker permissions."
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    try {
                        sh '''
                        # Use correct workspace path and check if pom.xml exists
                        if [ -f "${WORKSPACE}/pom.xml" ]; then
                            echo "Found pom.xml in workspace"
                            docker run --rm -v "${WORKSPACE}:/workspace" -w /workspace maven:3.9.9 \
                              mvn clean compile test -DskipTests=false
                        else
                            echo "pom.xml not found in workspace, listing files:"
                            ls -la "${WORKSPACE}"
                            exit 1
                        fi
                        '''
                    } catch (Exception e) {
                        echo "Build failed: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
        
        stage('Package') {
            steps {
                script {
                    try {
                        sh '''
                        # Package the application
                        if [ -f "${WORKSPACE}/pom.xml" ]; then
                            docker run --rm -v "${WORKSPACE}:/workspace" -w /workspace maven:3.9.9 \
                              mvn package -DskipTests=true
                        else
                            echo "pom.xml not found for packaging"
                            exit 1
                        fi
                        '''
                    } catch (Exception e) {
                        echo "Package failed: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
        
        stage('SonarQube') {
            steps {
                script {
                    try {
                        sh '''
                        # Run SonarQube analysis with network access
                        if [ -f "${WORKSPACE}/pom.xml" ]; then
                            docker run --rm --network host \
                              -v "${WORKSPACE}:/workspace" -w /workspace maven:3.9.9 \
                              mvn clean verify sonar:sonar \
                              -Dsonar.projectKey=${PROJECT_KEY} \
                              -Dsonar.projectName="${PROJECT_NAME}" \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.token=${SONAR_TOKEN}
                        else
                            echo "pom.xml not found for SonarQube analysis"
                            exit 1
                        fi
                        '''
                    } catch (Exception e) {
                        echo "SonarQube analysis failed: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                try {
                    // Clean up any remaining containers (if accessible)
                    sh '''
                    docker ps -aq --filter "name=maven-" | xargs -r docker stop || echo "No containers to stop"
                    docker ps -aq --filter "name=maven-" | xargs -r docker rm -f || echo "No containers to clean"
                    '''
                } catch (Exception e) {
                    echo "Cleanup warning: ${e.getMessage()}"
                }
            }
        }
        success {
            echo 'Pipeline completed successfully!'
            echo "SonarQube Analysis: ${SONAR_HOST_URL}/dashboard?id=${PROJECT_KEY}"
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
