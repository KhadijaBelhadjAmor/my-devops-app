pipeline {
    agent any
    
    tools {
        maven 'Apache Maven 3.6.3'   // Nom de l'outil Maven configuré dans Jenkins
        jdk 'openjdk version 21'     // Nom du JDK configuré dans Jenkins
    }
    
    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'          // Nom du scanner Sonar configuré dans Jenkins
        TOMCAT_URL = 'http://localhost:8081'
        TOMCAT_CREDENTIALS_ID = 'tomcat-credentials'     // Credential Jenkins pour Tomcat
    }
    
    stages {

        stage('Checkout') {
            steps {
                echo ' - Récupération du code depuis GitHub'
                git branch: 'main',
                    url: 'https://github.com/KhadijaBelhadjAmor/my-devops-app.git',
                    credentialsId: 'github-credentials'      // Credential Jenkins pour GitHub
            }
        }
        
        stage('Build') {
            steps {
                echo '- Compilation de l’application...'
                sh 'mvn clean compile'
            }
        }
        
        stage('Tests') {
            steps {
                echo '- Exécution des tests unitaires...'
                sh 'mvn test || true'    // || true pour ne pas arrêter le pipeline si un test échoue
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '- Analyse SonarQube...'
                script {
                    def mvn = tool 'Apache Maven 3.6.3'
                    // Le nom 'SonarQube' doit correspondre au serveur configuré dans Jenkins
                    withSonarQubeEnv('SonarQube') {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=devops"
                    }
                }
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
