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
    }
}
