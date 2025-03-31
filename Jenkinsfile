pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
    }
    environment{
        DOCKER_IMAGE = 'boardgameapp'
        DOCKER_TAG = 'latest'  // Static tag name
        DOCKER_HUB_USER = 'dockerr2021'
    }

    stages{
        stage("Checkout"){
            steps {
                checkout scm
            }
        }

        stage("Maven Build"){
            steps{
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis'){
            steps {
                withSonarQubeEnv('Sonar'){
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

        stage("Build Docker Image"){
            steps{
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage("Push Docker Image to Docker Hub"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]){
                        sh '''
                        echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USER" --password-stdin
                        docker tag "${DOCKER_IMAGE}:${DOCKER_TAG}" "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        docker push "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        '''
                    }
                }
            }
        }
    }
}
