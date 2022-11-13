pipeline {
    agent any

    stages {
        stage('checkout') {
            steps {
                git branch: 'development', credentialsId: '2ea143eb-76aa-488a-8239-561e3baf6c6a', url: 'https://github.com/Habeebasyad/hackathon-starter.git'
                
            }
            stage('ExecuteSonarQubeReport'){
nodejs(nodeJSInstallationName: 'nodejs12.0'){
sh "npm run sonar"
        }
    }
}
