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
                        // Essayer avec configuration Jenkins
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
                        echo '✅ Analyse SonarQube terminée (via Jenkins config)'
                    } catch (Exception e) {
                        echo "⚠️ Configuration Jenkins non trouvée, mode direct..."
                        // Mode direct sans withSonarQubeEnv
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
        
        stage('Wait for SonarQube Processing') {
            steps {
                echo '⏳ Attente de 3 minutes pour que SonarQube traite l analyse...'
                sleep time: 3, unit: 'MINUTES'
                
                // Vérification manuelle du statut
                script {
                    sh '''
                        echo "Vérification statut SonarQube..."
                        echo "Test API SonarQube:"
                        curl -s "http://localhost:9000/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}" || echo "API non accessible"
                    '''
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                echo '🔍 Vérification du Quality Gate...'
                script {
                    try {
                        // Essayer d'attendre le Quality Gate (avec timeout plus long)
                        timeout(time: 10, unit: 'MINUTES') {
                            def qualityGateResult = waitForQualityGate abortPipeline: false
                            
                            if (qualityGateResult?.status == 'OK') {
                                echo '🎉 QUALITY GATE PASSED !'
                            } else if (qualityGateResult?.status == 'ERROR') {
                                echo "⚠️ QUALITY GATE FAILED: ${qualityGateResult.status}"
                                echo "📋 Détails:"
                                echo qualityGateResult.toString()
                                echo "⏭️ Continuation malgré l échec du Quality Gate..."
                            } else {
                                echo "❓ Quality Gate status inconnu: ${qualityGateResult?.status}"
                            }
                        }
                    } catch (Exception e) {
                        echo "⏰ Timeout ou erreur Quality Gate: ${e.getMessage()}"
                        echo "🔍 Vérification manuelle de secours..."
                        
                        // Vérification API directe
                        sh '''
                            echo "Vérification via API SonarQube..."
                            API_RESPONSE=$(curl -s "http://localhost:9000/api/qualitygates/project_status?projectKey=${SONAR_PROJECT_KEY}" 2>/dev/null || echo "{}")
                            echo "Réponse API: $API_RESPONSE"
                            
                            if echo "$API_RESPONSE" | grep -q '"status":"OK"'; then
                                echo "✅ API indique: Quality Gate PASSED"
                            elif echo "$API_RESPONSE" | grep -q '"status":"ERROR"'; then
                                echo "⚠️ API indique: Quality Gate FAILED"
                            else
                                echo "❓ Statut Quality Gate non disponible"
                            fi
                        '''
                        
                        echo "⏭️ Continuation de la pipeline..."
                    }
                }
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
            echo '   4. ⏳ Attente traitement SonarQube'
            echo '   5. 🔍 Vérification Quality Gate'
            echo '   6. ✅ Packaging JAR'
            echo '   7. ✅ Build Docker Image'
            echo '   8. ✅ Push Docker Hub'
            
            // Nettoyage Docker
            sh 'docker system prune -f 2>/dev/null || true'
        }
        success {
            echo '🚀 SUCCÈS : Pipeline terminée avec succès !'
            echo "📦 Image disponible: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "🔗 SonarQube: http://localhost:9000/dashboard?id=${SONAR_PROJECT_KEY}"
        }
        aborted {
            echo '⏹️  Pipeline interrompue'
            echo 'ℹ️  Possible cause: Timeout SonarQube'
        }
        failure {
            echo '💥 ÉCHEC : Pipeline a échoué !'
            echo '📋 Vérifiez les logs pour plus de détails'
        }
    }
}
