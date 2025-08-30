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
                        # Alternative approach: Use docker cp instead of volume mounts
                        echo "=== Host workspace contents ==="
                        ls -la "${WORKSPACE}"
                        
                        # Create a temporary container
                        docker create --name maven-build-temp -w /app maven:3.9.9 sleep 30
                        docker start maven-build-temp
                        
                        # Copy files to container
                        docker cp "${WORKSPACE}/." maven-build-temp:/app/
                        
                        # Verify files are copied
                        docker exec maven-build-temp sh -c "echo 'Container contents:' && ls -la /app && echo 'pom.xml check:' && ls -la /app/pom.xml"
                        
                        # Run Maven build
                        docker exec maven-build-temp mvn clean compile test -DskipTests=false
                        
                        # Cleanup
                        docker stop maven-build-temp
                        docker rm maven-build-temp
                        '''
                    } catch (Exception e) {
                        sh '''
                        # Cleanup on error
                        docker stop maven-build-temp 2>/dev/null || true
                        docker rm maven-build-temp 2>/dev/null || true
                        '''
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
                        # Create container for packaging
                        docker create --name maven-package-temp -w /app maven:3.9.9 sleep 30
                        docker start maven-package-temp
                        
                        # Copy files to container
                        docker cp "${WORKSPACE}/." maven-package-temp:/app/
                        
                        # Run Maven package
                        docker exec maven-package-temp mvn package -DskipTests=true
                        
                        # Copy target folder back to workspace
                        mkdir -p "${WORKSPACE}/target" || true
                        docker cp maven-package-temp:/app/target/. "${WORKSPACE}/target/" || echo "No target directory to copy"
                        
                        # Cleanup
                        docker stop maven-package-temp
                        docker rm maven-package-temp
                        '''
                    } catch (Exception e) {
                        sh '''
                        # Cleanup on error
                        docker stop maven-package-temp 2>/dev/null || true
                        docker rm maven-package-temp 2>/dev/null || true
                        '''
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
                        # Create container for SonarQube analysis with network access
                        docker create --name maven-sonar-temp --network host -w /app maven:3.9.9 sleep 30
                        docker start maven-sonar-temp
                        
                        # Copy files to container
                        docker cp "${WORKSPACE}/." maven-sonar-temp:/app/
                        
                        # Run SonarQube analysis
                        docker exec maven-sonar-temp mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=${PROJECT_KEY} \
                          -Dsonar.projectName="${PROJECT_NAME}" \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.token=${SONAR_TOKEN}
                        
                        # Cleanup
                        docker stop maven-sonar-temp
                        docker rm maven-sonar-temp
                        '''
                    } catch (Exception e) {
                        sh '''
                        # Cleanup on error
                        docker stop maven-sonar-temp 2>/dev/null || true
                        docker rm maven-sonar-temp 2>/dev/null || true
                        '''
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
                    // Clean up any remaining containers
                    sh '''
                    # Stop and remove any leftover containers
                    for container in maven-build-temp maven-package-temp maven-sonar-temp; do
                        docker stop $container 2>/dev/null || echo "Container $container not running"
                        docker rm $container 2>/dev/null || echo "Container $container not found"
                    done
                    
                    # Generic cleanup for any maven containers
                    docker ps -aq --filter "name=maven-" | xargs -r docker stop 2>/dev/null || echo "No maven containers to stop"
                    docker ps -aq --filter "name=maven-" | xargs -r docker rm -f 2>/dev/null || echo "No maven containers to clean"
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
