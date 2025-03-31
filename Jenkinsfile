pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
    }
    environment{
        DOCKER_IMAGE = 'boardgameapp'
        // Generate date + incremental suffix (e.g., "202503311", "202503312", ...)
        DOCKER_TAG = sh(
            script: '''
                DATE=$(date +%Y%m%d)
                # Find the highest existing number for today (e.g., 202503311 → 1, 202503312 → 2)
                MAX_NUM=$(docker images --filter=reference="${DOCKER_IMAGE}:${DATE}*" --format "{{.Tag}}" | \
                          awk -F"${DATE}" '{print \$2}' | \
                          sort -n | \
                          tail -n 1)
                # If no images exist for today, start with 1, otherwise increment
                if [ -z "$MAX_NUM" ]; then
                    echo "${DATE}1"
                else
                    echo "${DATE}$((MAX_NUM + 1))"
                fi
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
            steps{
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
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage("Push Docker Image to Docker Hub"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]){
                        sh '''
                        set +x  # Disable command echoing for security
                        echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USER" --password-stdin
                        docker tag "${DOCKER_IMAGE}:${DOCKER_TAG}" "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        docker push "${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        docker logout
                        '''
                    }
                }
            }
        }
    }
}
