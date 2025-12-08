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
                sh 'mvn clean compile'
                echo '✅ Build Maven réussi'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
                echo '✅ Tests exécutés'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Démarrage de l analyse SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            mvn sonar:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.projectName="Student Management App" \
                            -Dsonar.token=${SONAR_TOKEN}
                        '''
                    }
                }
                echo '✅ Analyse SonarQube terminée'
            }
        }
        
        stage('Quality Gate') {
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
                sh '''
                    docker build -t student-management:latest .
                    docker tag student-management:latest ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker images | grep student-management
                '''
                echo '✅ Image Docker créée'
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
                            docker logout
                        '''
                    }
                }
                echo '✅ Image poussée sur Docker Hub'
            }
        }
    }
    
    post {
        always {
            echo '🎉 PIPELINE CI/CD TERMINÉE ! 🎉'
            echo '📋 Résumé des étapes exécutées :'
            echo '   1. ✅ Checkout Git'
            echo '   2. ✅ Build Maven'
            echo '   3. ✅ Tests'
            echo '   4. ✅ Analyse SonarQube'
            echo '   5. ✅ Quality Gate'
            echo '   6. ✅ Packaging JAR'
            echo '   7. ✅ Build Docker Image'
            echo '   8. ✅ Push Docker Hub'
        }
        success {
            echo '🚀 SUCCÈS : Pipeline terminée avec succès !'
            // Vous pouvez ajouter des notifications ici
        }
        failure {
            echo '💥 ÉCHEC : Pipeline a échoué !'
            // Vous pouvez ajouter des notifications d'erreur ici
        }
    }
}
