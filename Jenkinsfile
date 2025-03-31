pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
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
        //stage('OWASP Scan') {
        //    steps {
                // Note: 'odcInstallation' is the correct parameter name
          //      dependencyCheck additionalArguments: '--scan target/ --format ALL --project "MyApp"', 
           //                   odcInstallation: 'OWASP'
               // dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
           // }
    //    }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') { // Matches Jenkins SonarQube server name
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
    }
}
