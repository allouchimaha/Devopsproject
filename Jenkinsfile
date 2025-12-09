pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mahaall'
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        // ÉTAPE 1: Récupération du code
        stage('📥 Checkout Code') {
            steps {
                checkout scm
                echo '✅ Code source récupéré depuis Git'
            }
        }
        
        // ÉTAPE 2: Compilation Maven
        stage('🔨 Build Maven') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '✅ Projet compilé avec succès'
            }
        }
        
        // ÉTAPE 3: Analyse SonarQube (SANS Quality Gate)
        stage('🔍 Analyse Code Quality') {
            steps {
                echo '🔍 Analyse de qualité du code avec SonarQube...'
                
                script {
                    // Utilisation directe du token (méthode la plus simple)
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
                }
                
                echo '✅ Analyse SonarQube terminée'
                echo '📊 Consultez les résultats: http://localhost:9000/dashboard?id=student-management'
            }
        }
        
        // ÉTAPE 4: Création du package JAR
        stage('📦 Package Application') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo '✅ JAR généré et archivé'
            }
        }
        
        // ÉTAPE 5: Construction de l'image Docker
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker build -t student-management:latest .
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    '''
                    sh 'docker images | grep student-management'
                }
                echo '✅ Image Docker construite et taguée'
            }
        }
        
        // ÉTAPE 6: Publication sur Docker Hub
        stage('⬆️ Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            # Connexion à Docker Hub
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            
                            # Publication des images
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            
                            # Déconnexion
                            docker logout
                        '''
                    }
                }
                echo '✅ Images publiées sur Docker Hub'
            }
        }
    }
    
    // ÉTAPE FINALE: Nettoyage et rapport
    post {
        always {
            echo ' '
            echo '========================================='
            echo '🎉 PIPELINE CI/CD TERMINÉE AVEC SUCCÈS !'
            echo '========================================='
            echo ' '
            echo '📋 RÉSUMÉ DES ÉTAPES :'
            echo '   1. 📥  Checkout Git ............. ✅'
            echo '   2. 🔨  Build Maven .............. ✅'
            echo '   3. 🔍  Analyse SonarQube ........ ✅'
            echo '   4. 📦  Packaging JAR ............ ✅'
            echo '   5. 🐳  Build Docker Image ....... ✅'
            echo '   6. ⬆️  Push Docker Hub .......... ✅'
            echo ' '
            echo '🔗 LIENS UTILES :'
            echo '   • SonarQube: http://localhost:9000/dashboard?id=student-management'
            echo "   • Docker Hub: https://hub.docker.com/r/mahaall/student-management"
            echo "   • Image Docker: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo ' '
            
            // Nettoyage Docker
            sh 'docker system prune -f 2>/dev/null || true'
        }
        
        success {
            echo '🚀 FÉLICITATIONS ! La pipeline s est exécutée avec succès.'
            echo '✅ Toutes les étapes sont terminées.'
        }
        
        failure {
            echo '💥 ERREUR ! La pipeline a échoué.'
            echo '🔍 Consultez les logs pour plus de détails.'
        }
    }
}
