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
        
        stage('🔨 Build Application') {
            steps {
                sh 'mvn clean compile -DskipTests'
                echo '✅ Application compilée (JDK 17)'
            }
        }
        
        stage('🔍 Code Quality Analysis') {
            steps {
                echo '📊 Analyse SonarQube...'
                // Variable différente pour éviter les conflits
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_SECRET')]) {
                    // Version SÉCURISÉE sans interpolation dangereuse
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName="Student Management App" \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=\${SONAR_SECRET} \
                        -DskipTests
                    """
                }
                echo '✅ Analyse SonarQube terminée'
            }
        }
        
        stage('📦 Create Package and Dockerfile') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                
                sh '''
                    echo "Création du Dockerfile pour JDK 17..."
                    cat > Dockerfile << 'EOF'
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF
                    
                    echo "✅ Dockerfile créé"
                    ls -la Dockerfile
                '''
                
                echo '✅ Package JAR créé et Dockerfile prêt'
            }
        }
        
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    echo '🏗️  Construction de l image Docker...'
                    
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
        
        stage('⬆️ Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Version sécurisée
                        sh '''
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            docker push ''' + "${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}" + '''
                            docker push ''' + "${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest" + '''
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
            echo ' '
            echo '========================================='
            echo '🎉 PIPELINE CI/CD TERMINÉE !'
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
        }
    }
}
