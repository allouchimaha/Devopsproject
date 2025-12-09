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
                echo '✅ Analyse terminée'
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
                        # SANS SUDO - après configuration des permissions
                        docker build -t student-management:latest .
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    '''
                    sh 'docker images | grep student-management'
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
                            # SANS SUDO
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            docker logout
                        '''
                    }
                }
                echo '✅ Images publiées sur Docker Hub'
            }
        }
    }
    
    post {
        always {
            echo '🎉 PIPELINE TERMINÉE !'
            echo "📦 Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
    }
}
