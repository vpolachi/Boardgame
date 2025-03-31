pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
    }
    environment{
        DOCKER_IMAGE = 'boardgameapp'
        // Generate date + sequential number (e.g., "202503311")
        DOCKER_TAG = sh(
            script: '''
            DATE=$(date +%Y%m%d)
            # Check local and remote images for existing tags
            LOCAL_TAGS=$(docker images --filter=reference="${DOCKER_IMAGE}:${DATE}*" --format "{{.Tag}}" | grep "^${DATE}[0-9]\\+$" || true)
            REMOTE_TAGS=$(curl -s "https://hub.docker.com/v2/repositories/${DOCKER_HUB_USER}/${DOCKER_IMAGE}/tags/" | jq -r '.results[].name' | grep "^${DATE}[0-9]\\+$" || true)
            
            # Combine and find the highest number
            ALL_TAGS=$(echo -e "$LOCAL_TAGS\\n$REMOTE_TAGS" | grep . | sort -n)
            MAX_NUM=$(echo "$ALL_TAGS" | tail -1 | sed "s/^${DATE}//")
            
            # If no tags exist for today, start with 1, otherwise increment
            [ -z "$MAX_NUM" ] && echo "${DATE}1" || echo "${DATE}$((MAX_NUM + 1))"
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
