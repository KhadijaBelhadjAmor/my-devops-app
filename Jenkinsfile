pipeline {
    agent any
    
    tools {
        maven 'Apache Maven 3.6.3'
        jdk 'openjdk version 21'
    }
    
    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        TOMCAT_URL = 'http://localhost:8081'
        TOMCAT_CREDENTIALS_ID = 'tomcat-credentials'
    }
    
    stages {

        stage('Checkout') {
            steps {
                echo ' - Recupération du code depuis GitHub'
                git branch: 'main',
                    url: 'https://github.com/KhadijaBelhadjAmor/my-devops-app.git',
                    credentialsId: 'github-credentials'
            }
        }
        
        stage('Build') {
            steps {
                echo '- Création de l’application...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Tests') {
            steps {
                echo '- Exécution des tests unitaires...'
                sh 'mvn test|| true'
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
    def mvn = tool 'Default Maven';
    withSonarQubeEnv() {
      sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=devops"
    }
  }
        
        stage('Package') {
            steps {
                echo '- Création du package WAR...'
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                echo '- Déploiement sur Tomcat...'
                deploy adapters: [tomcat9(credentialsId: TOMCAT_CREDENTIALS_ID, path: '', url: TOMCAT_URL)],
                       contextPath: 'my-devops-app',
                       war: 'target/*.war'
            }
        }
    }
    
    post {

        success {
            echo '- Pipeline exécuté avec succès!'
        }

        failure {
            echo '- Pipeline a échoué!'
        }

        always {
            echo '- Fin de l’exécution du pipeline'
        }
    }
}
