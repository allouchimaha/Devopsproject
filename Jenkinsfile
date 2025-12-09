pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mahaall'
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('📥 Checkout Code') {
            steps {
                checkout scm
                echo '✅ Code récupéré depuis Git'
            }
        }
        
        stage('🔨 Build Maven') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '✅ Projet compilé'
            }
        }
        
        stage('🔍 Analyse SonarQube') {
            steps {
                echo '🔍 Analyse qualité avec SonarQube...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.projectName="Student Management App" \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -DskipTests
                    '''
                }
                echo '✅ Analyse terminée: http://localhost:9000/dashboard?id=student-management'
            }
        }
        
        stage('📦 Package Application') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo '✅ JAR généré et archivé'
            }
        }
        
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    sh '''
                        # Nettoyage avant build
                        sudo docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} 2>/dev/null || true
                        sudo docker rmi student-management:latest 2>/dev/null || true
                        
                        # Build image
                        sudo docker build -t student-management:latest .
                        sudo docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        sudo docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    '''
                    sh 'sudo docker images | grep student-management'
                }
                echo '✅ Image Docker construite'
            }
        }
        
        stage('⬆️ Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            # Push vers Docker Hub
                            echo ${DOCKER_PASS} | sudo docker login -u ${DOCKER_USER} --password-stdin
                            sudo docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            sudo docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            sudo docker logout
                        '''
                    }
                }
                echo '✅ Images publiées sur Docker Hub'
            }
        }
    }
    
    post {
        always {
            echo ' '
            echo '========================================='
            echo '🎉 PIPELINE CI/CD TERMINÉE !'
            echo '========================================='
            echo ' '
            echo "📦 Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo '🔗 SonarQube: http://localhost:9000/dashboard?id=student-management'
            echo '🐳 Docker Hub: https://hub.docker.com/r/mahaall/student-management'
            echo ' '
            
            sh 'sudo docker system prune -f 2>/dev/null || true'
        }
    }
}
