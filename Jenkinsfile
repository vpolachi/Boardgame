pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
    }
    environment{
        DOCKER_IMAGE = 'boardgameapp'
        DOCKER_TAG = sh(script: '''DATE=$(date +%Y%m%d)
            LOCAL_TAGS=$(docker images --filter=reference="${DOCKER_IMAGE}:${DATE}*" --format "{{.Tag}}" | grep "^${DATE}[0-9]\\+$" || true)
            REMOTE_TAGS=$(curl -s "https://hub.docker.com/v2/repositories/${DOCKER_HUB_USER}/${DOCKER_IMAGE}/tags/" | jq -r '.results[].name' | grep "^${DATE}[0-9]\\+$" || true)
            ALL_TAGS=$(echo -e "$LOCAL_TAGS\\n$REMOTE_TAGS" | grep . | sort -n)
            MAX_NUM=$(echo "$ALL_TAGS" | tail -1 | sed "s/^${DATE}//")
            [ -z "$MAX_NUM" ] && echo "${DATE}1" || echo "${DATE}$((MAX_NUM + 1))"''', 
            returnStdout: true).trim()
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
                    sh "mvn sonar:sonar -Dsonar.projectKey=projkey -Dsonar.projectName='projdispname' -Dsonar.java.binaries=target/classes -Dsonar.sources=src/main/java"
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
                withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]){
                    sh 'echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USER" --password-stdin'
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }
}
