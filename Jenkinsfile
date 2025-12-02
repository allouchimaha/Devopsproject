pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t student-management:latest .
                    docker images student-management
                '''
            }
        }
        stage('SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=student-management -Dsonar.token=$SONAR_TOKEN'
                }
            }
        }
    }
    post {
        always {
            echo '🎉 PIPELINE DEVOPS 5/5 TERMINÉE! 🎉'
            echo '✅ Build → Test → Package → Docker → SonarQube'
        }
    }
}
