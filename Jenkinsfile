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
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SAST - SonarQube') {
            steps {
                echo '- Analyse de code avec SonarQube...'
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=my-devops-app -Dsonar.projectName="My DevOps App"'
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
            emailext(
                subject: "SUCCESS: Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """
Le pipeline CI/CD a réussi.

Détails :
- Job : ${env.JOB_NAME}
- Build : ${env.BUILD_NUMBER}
- URL du build : ${env.BUILD_URL}
- Application déployée : http://${env.JENKINS_URL}/my-devops-app
                """,
                to: "votre.email@example.com"
            )
        }

        failure {
            echo '- Pipeline a échoué!'
            emailext(
                subject: "FAILED: Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """
Le pipeline CI/CD a échoué.

Détails :
- Job : ${env.JOB_NAME}
- Build : ${env.BUILD_NUMBER}
- URL du build : ${env.BUILD_URL}

Consultez les logs Jenkins pour plus de détails.
                """,
                to: "votre.email@example.com"
            )
        }

        always {
            echo '- Fin de l’exécution du pipeline'
            cleanWs()
        }
    }
}
