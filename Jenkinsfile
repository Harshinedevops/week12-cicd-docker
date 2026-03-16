pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "harshine10/week12-cicd"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "========== Pulling Code from GitHub =========="
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo "========== Running Tests =========="
                sh 'pip3 install pytest'
                sh 'python3 -m pytest test_app.py -v'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                echo "========== Running SonarCloud Analysis =========="
                withSonarQubeEnv('SonarCloud') {
                    sh """
                    /opt/sonar-scanner/bin/sonar-scanner \
                    -Dsonar.projectKey=Harshine_week12-cicd-docker \
                    -Dsonar.organization=Harshine \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=https://sonarcloud.io \
                    -Dsonar.login=\$SONAR_AUTH_TOKEN
                    """
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "========== Building Docker Image =========="
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "========== Pushing Docker Image =========="
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy Container') {
            steps {
                echo "========== Deploying Docker Container =========="

                sh "docker stop week12-app || true"
                sh "docker rm week12-app || true"

                sh "docker run -d -p 8888:8888 --name week12-app ${DOCKER_IMAGE}:latest"

                echo "Application deployed successfully!"
            }
        }
    }

    post {
        success {
            echo "========== PIPELINE COMPLETED SUCCESSFULLY =========="
        }
        failure {
            echo "========== PIPELINE FAILED =========="
        }
    }
}
