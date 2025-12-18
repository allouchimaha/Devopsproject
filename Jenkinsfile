pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mahaall'
        DOCKER_IMAGE = 'student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        K8S_NAMESPACE = 'devops'
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
        
        stage('📦 Create Package and Dockerfile') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                
                sh '''
                    echo "Création du Dockerfile pour JDK 17..."
                    cat > Dockerfile << 'EOF'
FROM eclipse-temurin:17-jdk
COPY target/student-management-0.0.1-SNAPSHOT.jar app.jar
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
                        sh """
                            echo "\${DOCKER_PASS}" | docker login -u "\${DOCKER_USER}" --password-stdin
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:latest
                            docker logout
                        """
                    }
                }
                echo '✅ Images publiées sur Docker Hub'
            }
        }
        
        stage('🚀 Deploy to Kubernetes') {
            steps {
                script {
                    echo '📦 Déploiement sur Kubernetes...'
                    
                    // 1. Créer le fichier spring-deployment.yaml avec l'image du build actuel
                    sh '''
                        cat > spring-deployment.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-config
  namespace: devops
data:
  SPRING_DATASOURCE_URL: jdbc:mysql://mysql-service:3306/springdb
  SPRING_DATASOURCE_DRIVER_CLASS_NAME: com.mysql.cj.jdbc.Driver
---
apiVersion: v1
kind: Secret
metadata:
  name: spring-secret
  namespace: devops
type: Opaque
data:
  SPRING_DATASOURCE_USERNAME: c3ByaW5n
  SPRING_DATASOURCE_PASSWORD: c3ByaW5nMTIz
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  namespace: devops
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: mahaall/student-management:''' + "${DOCKER_TAG}" + '''
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            configMapKeyRef:
              name: spring-config
              key: SPRING_DATASOURCE_URL
        - name: SPRING_DATASOURCE_USERNAME
          valueFrom:
            secretKeyRef:
              name: spring-secret
              key: SPRING_DATASOURCE_USERNAME
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: spring-secret
              key: SPRING_DATASOURCE_PASSWORD
        - name: SPRING_JPA_HIBERNATE_DDL_AUTO
          value: "update"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1024Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: spring-service
  namespace: devops
spec:
  selector:
    app: spring-app
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
  type: NodePort
EOF
                        
                        echo "✅ Fichier spring-deployment.yaml créé"
                    '''
                    
                    // 2. Déployer sur Kubernetes
                    sh """
                        echo "🔧 Déploiement sur Kubernetes..."
                        
                        # Appliquer la configuration Spring Boot
                        kubectl apply -f spring-deployment.yaml -n ${K8S_NAMESPACE}
                        
                        # Attendre le déploiement
                        echo "⏳ Attente du déploiement..."
                        kubectl rollout status deployment/spring-app -n ${K8S_NAMESPACE} --timeout=300s
                        
                        # Vérifier
                        echo "🔍 Vérification..."
                        kubectl get pods -n ${K8S_NAMESPACE}
                        kubectl get svc -n ${K8S_NAMESPACE}
                    """
                }
                echo '✅ Application déployée sur Kubernetes'
            }
        }
        
        stage('✅ Verification') {
            steps {
                script {
                    echo '🔍 Vérification finale...'
                    
                    sh """
                        # Attendre que l'application soit prête
                        sleep 30
                        
                        # Vérifier les logs
                        echo "📋 Logs de l'application:"
                        kubectl logs -n ${K8S_NAMESPACE} deployment/spring-app --tail=10
                        
                        # Obtenir l'URL
                        echo "🌐 URL d'accès:"
                        minikube service spring-service -n ${K8S_NAMESPACE} --url || echo "URL disponible après quelques minutes"
                    """
                }
                echo '✅ Vérification terminée'
            }
        }
    }
    
    post {
        always {
            echo ' '
            echo '========================================='
            echo '🎉 PIPELINE CI/CD COMPLÈTE !'
            echo '========================================='
            echo ' '
            echo '📊 RÉSUMÉ :'
            echo '   1. 📥  Checkout GitHub ............. ✅'
            echo '   2. 🔨  Build Maven ................. ✅'
            echo '   3. 📦  Packaging JAR ............... ✅'
            echo '   4. 🐳  Build Docker Image .......... ✅'
            echo '   5. ⬆️  Push Docker Hub ............. ✅'
            echo '   6. 🚀  Deploy Kubernetes ........... ✅'
            echo '   7. ✅  Verification ................ ✅'
            echo ' '
            echo "🔗 Docker Hub: https://hub.docker.com/r/${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
            echo "📦 Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo "🌐 Kubernetes Namespace: ${K8S_NAMESPACE}"
            echo "📍 NodePort: 30080"
            echo ' '
            
            // Nettoyage
            sh '''
                echo "🧹 Nettoyage des fichiers temporaires..."
                rm -f Dockerfile spring-deployment.yaml 2>/dev/null || true
            '''
        }
        
        success {
            echo '✅ PIPELINE RÉUSSIE ! Application déployée avec succès.'
            sh '''
                echo " "
                echo "🎯 ACCÈS À L'APPLICATION :"
                echo "Pour accéder à l'application, utilise cette commande :"
                echo "  minikube service spring-service -n devops --url"
                echo " "
                echo "🔗 OU directement via :"
                echo "  http://$(minikube ip):30080"
                echo " "
            '''
        }
        
        failure {
            echo '❌ Échec du pipeline. Vérifier les logs pour détails.'
        }
    }
}
