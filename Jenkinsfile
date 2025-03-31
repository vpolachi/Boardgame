pipeline {
    agent any
    tools {
        maven 'maven3.9.9'
    }
    environment {
        DOCKER_IMAGE = 'boardgameapp'
        DOCKER_TAG_DATE = sh(script: 'date +%Y%m%d', returnStdout: true).trim()  // Only date (e.g., "20240316")
        DOCKER_HUB_USER = 'dockerr2021'
    }

    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh """
                    mvn sonar:sonar \
                      -Dsonar.projectKey=projkey \
                      -Dsonar.projectName='projdispname' \
                      -Dsonar.java.binaries=target/classes \
                      -Dsonar.sources=src/main/java
                    """
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    // Build with both date tag and latest tag
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG_DATE} -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage("Push Docker Image to Docker Hub") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]) {
                        sh '''
                        echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USER" --password-stdin
                        docker tag "${DOCKER_IMAGE}:${DOCKER_TAG_DATE}" "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG_DATE}"
                        docker tag "${DOCKER_IMAGE}:latest" "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:latest"
                        docker push "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG_DATE}"
                        docker push "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:latest"
                        '''
                    }
                }
            }
        }
    }
}
