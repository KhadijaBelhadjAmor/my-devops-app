pipeline {
    agent any
    
    tools {
        maven 'Maven-3.6.3'
        jdk 'JDK-21'
    }
    
    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'
        TOMCAT_URL = 'http://localhost:8081'
        TOMCAT_CREDENTIALS_ID = 'tomcat-credentials'
    }
    
    stages {
        // Stage 1: Checkout du code
        stage('Checkout') {
            steps {
                echo ' -Recuperation du code depuis GitHub'
                git branch: 'main', 
                    url: 'https://github.com/KhadijaBelhadjAmor/my-devops-app.git',
                    credentialsId: 'github-credentials'
            }
        }
        
        // Stage 2: Build de l'application
        stage('Build') {
            steps {
                echo '-Creation de lapplication...'
                sh 'mvn clean compile'
            }
        }
        
        // Stage 3: Tests unitaires
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
        
        // Stage 4: SAST avec SonarQube
        stage('SAST - SonarQube') {
            steps {
                echo '- Analyse de code avec SonarQube...'
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=my-devops-app -Dsonar.projectName="My DevOps App"'
                }
            }
        }
        
        // Stage 5: Packaging
        stage('Package') {
            steps {
                echo '- Creation du package WAR...'
                sh 'mvn package -DskipTests'
            }
        }
        
        // Stage 6: Deploiement sur Tomcat
        stage('Deploy to Tomcat') {
            steps {
                echo '- Deploiement sur Tomcat...'
                deploy adapters: [tomcat9(credentialsId: TOMCAT_CREDENTIALS_ID, path: '', url: TOMCAT_URL)], 
                      contextPath: 'my-devops-app',
                      war: 'target/*.war'
            }
        }
    }
    
    post {
        success {
            echo '- Pipeline exécuté avec succès!'
            emailext (
                subject: "SUCCESS: Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                
                body: "Le pipeline CI/CD a réussi. Consulter: ${env.BUILD_URL}"
                
                Details:
                - Job: ${env.JOB_NAME}
                - Build: ${env.BUILD_NUMBER}
                - URL: ${env.BUILD_URL}
                - Application: http://${env.JENKINS_URL}/my-devops-app
                """,
                to: "votre.email@example.com"
            )
        }
        failure {
            echo '- Pipeline a échoué!'
            emailext (
                subject: "FAILED: Pipeline '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
               
                body: "Le pipeline CI/CD a échoué. Consulter: ${env.BUILD_URL}"
                
                Details:
                - Job: ${env.JOB_NAME}
                - Build: ${env.BUILD_NUMBER}
                - URL: ${env.BUILD_URL}
                - Consulter les logs pour plus de détails
                """,
                to: "votre.email@example.com"
            )
        }
        always {
            echo '-Fin de l exécution du pipeline'
            cleanWs()
        }
    }
}
