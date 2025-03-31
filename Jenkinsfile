pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
    }
    environment{
        DOCKER_IMAGE = 'boardgameapp'
        // Generate date + incremental suffix (e.g., "20250331-1")
        DOCKER_TAG = sh(
            script: '''
            DATE=$(date +%Y%m%d)
            COUNT=$(docker images --filter=reference="${DOCKER_IMAGE}:${DATE}-*" --format "{{.Tag}}" | wc -l)
            echo "${DATE}-$((COUNT + 1))"
            ''', 
            returnStdout: true
        ).trim()
        DOCKER_HUB_USER = 'dockerr2021'
    }

    stages{
        stage("Checkout"){
            steps{
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
                script{
                    // Build with only the date-incremental tag
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
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
