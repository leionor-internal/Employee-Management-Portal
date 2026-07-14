pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        DOCKER_IMAGE = "akashms54/employee-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                    -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Remove Old Container') {
            steps {
                sh 'docker compose down || true'
            }
        }

        stage('Run New Container') {
            steps {
                sh '''
                    docker compose pull
                    docker compose up -d --no-build
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 20
                    curl -f http://localhost:8085/employees
                '''
            }
        }
    }

    post {
        success {
            echo 'Employee Management Pipeline SUCCESS!'
        }

        failure {
            echo 'Pipeline FAILED. Check logs.'
        }

        always {
            echo 'Pipeline finished.'
        }
    }
}
