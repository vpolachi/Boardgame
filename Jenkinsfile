pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
    }
    environment{
    DOCKER_IMAGE = 'boardgameapp'
    DOCKER_TAG = 'cicd1.0'
    DOCKER_HUB_USER = 'dockerr2021' // Replace with your actual Docker Hub username
}

    stages{
        stage("Checkout"){
            steps{
                checkout scm
            }
        }
         stage("Increment Docker Tag"){
            steps{
                script{
                    // Fetch the latest tag from Docker Hub
                    def lastTag = sh(script: "curl -s \"https://hub.docker.com/v2/repositories/$DOCKER_HUB_USER/$DOCKER_IMAGE/tags?page_size=100\" | jq -r '.results[].name' | grep -E '^cicd[0-9]+\\.[0-9]+$' | sort -V | tail -n 1", returnStdout: true).trim()
                    
                    if (!lastTag){
                        lastTag = "cicd1.0"  // Default if no tags exist
                    }
                    
                    // Split the tag and increment the last part
                    def tagParts = lastTag.tokenize('.')
                    def newTag = "cicd" + tagParts[0][-1] + "." + (tagParts[1].toInteger() + 1)
                    
                    // Set the new tag in the environment variable
                    env.DOCKER_TAG = newTag
                }
            }
        }
        stage("Maven Build"){
            steps{
                sh 'mvn clean package'
            }
        }
        //stage('OWASP Scan') {
        //    steps {
                // Note: 'odcInstallation' is the correct parameter name
          //      dependencyCheck additionalArguments: '--scan target/ --format ALL --project "MyApp"', 
           //                   odcInstallation: 'OWASP'
               // dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
           // }
    //    }
        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('Sonar'){ // Matches Jenkins SonarQube server name
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
            def imageName = "boardgameapp"  
            def imageTag = "cicd1.0"  

            sh "docker build -t $imageName:$imageTag ."
        }
    }
}
         stage("Push Docker Image to Docker Hub"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]){
                        sh """
                        echo $DOCKER_HUB_TOKEN | docker login -u $DOCKER_HUB_USER --password-stdin
                        docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_HUB_USER/$DOCKER_IMAGE:$DOCKER_TAG
                        docker push $DOCKER_HUB_USER/$DOCKER_IMAGE:$DOCKER_TAG
                        """
                    }
                }
            }

}
}
}
