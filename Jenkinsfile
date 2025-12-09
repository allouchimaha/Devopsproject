pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mahaall'
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        SONAR_PROJECT_KEY = 'student-management'
        SONAR_HOST_URL = 'http://localhost:9000'
    }
    
    stages {
        // ÉTAPE 1: Récupération du code
        stage('📥 Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/maha']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/allouchimaha/Devopsproject.git',
                        credentialsId: 'github-jenkins-token'
                    ]]
                ])
                echo '✅ Code source récupéré'
            }
        }
        
        // ÉTAPE 2: Compilation
        stage('🔨 Build Application') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '✅ Application compilée'
            }
        }
        
        // ÉTAPE 3: Analyse SonarQube
        stage('🔍 Code Quality Analysis') {
            steps {
                echo '📊 Analyse SonarQube...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="Student Management App" \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -DskipTests
                    """
                }
                echo '✅ Analyse SonarQube terminée'
            }
        }
        
        // ÉTAPE 4: Packaging
        stage('📦 Create Package') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo '✅ Package JAR créé'
            }
        }
        
        // ÉTAPE 5: Build Docker Image (SIMPLIFIÉE)
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    echo '🏗️  Construction de l image Docker...'
                    
                    // Simple et direct - Docker fonctionne maintenant
                    sh """
                        docker build -t student-management:latest .
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    """
                    
                    sh 'docker images | grep student-management'
                }
                echo '✅ Image Docker construite'
            }
        }
        
        // ÉTAPE 6: Push to Docker Hub (SIMPLIFIÉE)
        stage('⬆️ Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            # Authentification et push
                            echo \${DOCKER_PASS} | docker login -u \${DOCKER_USER} --password-stdin
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            docker logout
                        """
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
            echo '🎉 PIPELINE CI/CD TERMINÉE AVEC SUCCÈS !'
            echo '========================================='
            echo ' '
            echo '📊 RÉSUMÉ :'
            echo '   1. 📥  Checkout GitHub ............. ✅'
            echo '   2. 🔨  Build Maven ................. ✅'
            echo '   3. 🔍  Analyse SonarQube ........... ✅'
            echo '   4. 📦  Packaging JAR ............... ✅'
            echo '   5. 🐳  Build Docker Image .......... ✅'
            echo '   6. ⬆️  Push Docker Hub ............. ✅'
            echo ' '
            echo "🔗 SonarQube: ${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}"
            echo "🔗 Docker Hub: https://hub.docker.com/r/${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
            echo "📦 Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo ' '
            
            // Nettoyage
            sh 'docker system prune -f 2>/dev/null || true'
        }
    }
}
