pipeline {
    agent any

    stages {
        // 1. Code लिने
        stage('Get Code') {
            steps {
                checkout scm
            }
        }

        // 2. WAR बनाउने
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 3. Docker Image बनाउने
        stage('Docker Build') {
            steps {
                sh "docker build -t suresh53/esewa:${BUILD_NUMBER} ."
                sh "docker tag suresh53/esewa:${BUILD_NUMBER} suresh53/esewa:latest"
            }
        }

        // 4. Docker Hub मा Push गर्ने
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push suresh53/esewa:${BUILD_NUMBER}"
                    sh "docker push suresh53/esewa:latest"
                }
            }
        }

        // 5. Kubernetes मा Deploy गर्ने
        stage('Deploy') {
            steps {
                sh "sed 's|suresh53/esewa:.*|suresh53/esewa:${BUILD_NUMBER}|' deployment.yaml | /usr/local/bin/kubectl apply -f -"
                sh "/usr/local/bin/kubectl apply -f service.yaml"
            }
        }

        // 6. Check गर्ने
        stage('Check') {
            steps {
                sh '/usr/local/bin/kubectl get pods'
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}