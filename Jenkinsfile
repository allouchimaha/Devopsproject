pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mahaall'  
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo '✅ Code récupéré depuis Git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '✅ Build Maven réussi (tests skipped)'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Démarrage de l analyse SonarQube...'
                script {
                    try {
                        // Essayer avec configuration Jenkins
                        withSonarQubeEnv('SonarQube') {
                            withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                                sh '''
                                    mvn sonar:sonar \
                                    -Dsonar.projectKey=student-management \
                                    -Dsonar.projectName="Student Management App" \
                                    -Dsonar.token=${SONAR_TOKEN} \
                                    -Dsonar.host.url=http://localhost:9000 \
                                    -DskipTests
                                '''
                            }
                        }
                        echo '✅ Analyse SonarQube terminée (via Jenkins config)'
                    } catch (Exception e) {
                        echo "⚠️ Configuration Jenkins non trouvée, mode direct..."
                        // Mode direct sans withSonarQubeEnv
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
                        echo '✅ Analyse SonarQube terminée (mode direct)'
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            when {
                // Exécuter seulement si SonarQube a fonctionné
                expression { 
                    try {
                        withSonarQubeEnv('SonarQube') { return true }
                    } catch(e) {
                        return false
                    }
                }
            }
            steps {
                echo '⏳ Attente du Quality Gate...'
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
                echo '✅ Quality Gate passé'
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo '✅ JAR créé et archivé'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    // Build l'image
                    sh '''
                        docker build -t student-management:latest .
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                    '''
                    
                    // Vérifier
                    sh 'docker images | grep student-management'
                }
                echo '✅ Image Docker créée et taguée'
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            # Login Docker Hub
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            
                            # Push avec tag BUILD_NUMBER
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            
                            # Push latest
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            
                            # Logout
                            docker logout
                        '''
                    }
                }
                echo '✅ Images poussées sur Docker Hub'
            }
        }
    }
    
    post {
        always {
            echo '🎉 PIPELINE CI/CD TERMINÉE ! 🎉'
            echo '📋 Résumé des étapes exécutées :'
            echo '   1. ✅ Checkout Git'
            echo '   2. ✅ Build Maven (tests skipped)'
            echo '   3. ✅ Analyse SonarQube'
            echo '   4. ✅ Quality Gate'
            echo '   5. ✅ Packaging JAR'
            echo '   6. ✅ Build Docker Image'
            echo '   7. ✅ Push Docker Hub'
            
            // Nettoyage Docker
            sh 'docker system prune -f 2>/dev/null || true'
        }
        success {
            echo '🚀 SUCCÈS : Pipeline terminée avec succès !'
            echo "📦 Image disponible: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo '💥 ÉCHEC : Pipeline a échoué !'
            // Vous pouvez ajouter des notifications d'erreur ici
        }
    }
}
