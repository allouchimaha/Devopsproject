pipeline {
    agent any
    
    environment {
        // Configuration Docker
        DOCKER_REGISTRY = 'mahaall'
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        
        // Configuration SonarQube
        SONAR_PROJECT_KEY = 'student-management'
        SONAR_PROJECT_NAME = 'Student Management App'
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
                        credentialsId: 'github-jenkins-token'  // ← VOTRE NOUVEAU CREDENTIAL
                    ]]
                ])
                echo '✅ Code source récupéré depuis GitHub'
            }
        }
        
        // ÉTAPE 2: Compilation
        stage('🔨 Build Application') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '✅ Application compilée avec succès'
            }
        }
        
        // ÉTAPE 3: Analyse SonarQube
        stage('🔍 Code Quality Analysis') {
            steps {
                echo '📊 Analyse de qualité avec SonarQube...'
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="${SONAR_PROJECT_NAME}" \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN} \
                        -DskipTests
                    """
                }
                echo '✅ Analyse SonarQube terminée'
                echo "🔗 Dashboard: ${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}"
            }
        }
        
        // ÉTAPE 4: Packaging
        stage('📦 Create Package') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo '✅ Package JAR créé et archivé'
            }
        }
        
        // ÉTAPE 5: Build Docker Image
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    // Nettoyage des anciennes images locales
                    sh '''
                        docker rmi ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG} 2>/dev/null || true
                        docker rmi student-management:latest 2>/dev/null || true
                    '''
                    
                    // Construction de la nouvelle image
                    sh """
                        docker build -t student-management:latest .
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    """
                    
                    // Vérification
                    sh 'docker images | grep student-management'
                }
                echo '✅ Image Docker construite et taguée'
            }
        }
        
        // ÉTAPE 6: Push to Docker Hub
        stage('⬆️ Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            # Authentification Docker Hub
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            
                            # Publication des images
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            
                            # Déconnexion
                            docker logout
                        """
                    }
                }
                echo '✅ Images publiées sur Docker Hub'
                echo "🔗 https://hub.docker.com/r/${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
            }
        }
    }
    
    post {
        always {
            echo ' '
            echo '============================================'
            echo '🎉 PIPELINE CI/CD TERMINÉE AVEC SUCCÈS !'
            echo '============================================'
            echo ' '
            echo '📊 RÉSUMÉ DES ÉTAPES :'
            echo '   1. 📥  Checkout GitHub ............. ✅'
            echo '   2. 🔨  Build Maven ................. ✅'
            echo '   3. 🔍  Analyse SonarQube ........... ✅'
            echo '   4. 📦  Packaging JAR ............... ✅'
            echo '   5. 🐳  Build Docker Image .......... ✅'
            echo '   6. ⬆️  Push Docker Hub ............. ✅'
            echo ' '
            echo '🔗 LIENS :'
            echo "   • SonarQube: ${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}"
            echo "   • Docker Hub: https://hub.docker.com/r/${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
            echo "   • Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo ' '
            
            // Nettoyage
            sh 'docker system prune -f 2>/dev/null || true'
        }
        
        success {
            echo '🚀 FÉLICITATIONS ! Toutes les étapes sont terminées.'
        }
        
        failure {
            echo '💥 ERREUR ! Consultez les logs pour diagnostiquer.'
        }
    }
}
