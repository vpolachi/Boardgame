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
            def tagsJson = sh(script: """
                curl -s "https://hub.docker.com/v2/repositories/${DOCKER_HUB_USER}/${DOCKER_IMAGE}/tags/?page_size=100"
            """, returnStdout: true).trim()
            
            // Parse JSON and extract all tag names
            def tags = new groovy.json.JsonSlurper().parseText(tagsJson).results*.name
            
            // Filter tags starting with 'cicd' and get the highest number
            def lastTag = tags.findAll { it.startsWith('cicd') }
                             .collect { it.replaceAll(/^cicd/, '') }
                             .findAll { it.isNumber() }
                             .max { it.toInteger() }
            
            // Calculate new tag number
            def nextNumber = (lastTag ? lastTag.toInteger() : 0) + 1
            env.DOCKER_TAG = "cicd${nextNumber}"
            
            echo "Using Docker Tag: ${env.DOCKER_TAG}"
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
