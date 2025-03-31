pipeline{
    agent any
    tools{
        maven 'maven3.9.9'
        dependency-check 'OWASP'
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
        stage('OWASP'){
            steps{
                // Scan compiled artifacts in 'target/'
                dependencyCheck additionalArguments: '--scan target/ --format All', odcInstallation: 'OWASP'             
                // Publish results to Jenkins UI
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
    }
}
       
