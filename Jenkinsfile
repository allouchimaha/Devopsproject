pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mahaall'  
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        SONAR_PROJECT_KEY = 'student-management'
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
                        withSonarQubeEnv('SonarQube') {
                            withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                                sh '''
                                    mvn sonar:sonar \
                                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                    -Dsonar.projectName="Student Management App" \
                                    -Dsonar.token=${SONAR_TOKEN} \
                                    -Dsonar.host.url=http://localhost:9000 \
                                    -DskipTests
                                '''
                            }
                        }
                        echo '✅ Analyse SonarQube terminée'
                        echo "🔗 Consultez: http://localhost:9000/dashboard?id=${SONAR_PROJECT_KEY}"
                    } catch (Exception e) {
                        echo "⚠️ Configuration Jenkins non trouvée, mode direct..."
                        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                            sh '''
                                mvn sonar:sonar \
                                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
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
        
        // QUALITY GATE OPTIONNEL - DÉCOMMENTEZ SI BESOIN
        /*
        stage('Quality Gate (Optional)') {
            steps {
                echo '⏳ Vérification Quality Gate (3min max)...'
                script {
                    try {
                        timeout(time: 3, unit: 'MINUTES') {
                            def qg = waitForQualityGate abortPipeline: false
                            if (qg?.status == 'OK') {
                                echo '🎉 QUALITY GATE PASSED !'
                            } else {
                                echo "⚠️ Quality Gate: ${qg?.status ?: 'Non disponible'}"
                            }
                        }
                    } catch (Exception e) {
                        echo "⏰ Timeout Quality Gate - skip"
                    }
                }
            }
        }
        */
        
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
                    sh '''
                        docker build -t student-management:latest .
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                        docker images | grep student-management
                    '''
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
                            echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
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
            echo '   4. ⏭️ Quality Gate skipped (optionnel)'
            echo '   5. ✅ Packaging JAR'
            echo '   6. ✅ Build Docker Image'
            echo '   7. ✅ Push Docker Hub'
            echo ''
            echo "🔗 SonarQube Dashboard: http://localhost:9000/dashboard?id=${SONAR_PROJECT_KEY}"
            echo "📦 Docker Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            
            sh 'docker system prune -f 2>/dev/null || true'
        }
        success {
            echo '🚀 SUCCÈS : Pipeline terminée avec succès !'
        }
        failure {
            echo '💥 ÉCHEC : Pipeline a échoué !'
        }
    }
}
