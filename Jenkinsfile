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
                echo '✅ Application compilée'
            }
        }
        
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
        
        stage('📦 Create Package and Dockerfile') {
            steps {
                // 1. Créer le JAR
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                
                // 2. Créer Dockerfile s'il n'existe pas
                sh '''
                    echo "Vérification du Dockerfile..."
                    if [ ! -f Dockerfile ]; then
                        echo "Création du Dockerfile..."
                        cat > Dockerfile << 'EOF'
FROM openjdk:11-jre-slim
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF
                        echo "✅ Dockerfile créé"
                    else
                        echo "✅ Dockerfile existe déjà"
                    fi
                    
                    # Afficher le Dockerfile pour vérification
                    echo "Contenu du Dockerfile:"
                    cat Dockerfile
                    echo ""
                    echo "Fichiers présents:"
                    ls -la
                '''
                
                echo '✅ Package JAR créé et Dockerfile vérifié'
            }
        }
        
        stage('🐳 Build Docker Image') {
            steps {
                script {
                    echo '🏗️  Construction de l image Docker...'
                    
                    // Afficher les fichiers avant build
                    sh '''
                        echo "Fichiers dans le répertoire:"
                        ls -la
                        echo ""
                        echo "Contenu de Dockerfile:"
                        cat Dockerfile || echo "Dockerfile non trouvé"
                    '''
                    
                    // Build l'image
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
                        sh """
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
            echo '🎉 PIPELINE CI/CD TERMINÉE !'
            echo '========================================='
            echo ' '
            echo "🔗 SonarQube: ${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}"
            echo "🔗 Docker Hub: https://hub.docker.com/r/${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
            echo "📦 Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo ' '
        }
    }
}
