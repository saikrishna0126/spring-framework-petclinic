pipeline {
    agent any
    tools {
        maven 'maven' // Make sure Maven tool is configured in Jenkins
        // You can define other tools here as needed
    }
    environment {
        // SonarQube environment variables
        SONAR_SCANNER='C:\\Sonarscanner\\sonar-scanner-5.0.1.3006-windows\\bin\\sonar-scanner.bat'
        SONAR_URL='http://localhost:9000'
        SONAR_PROJECTKEY='java'
        SONAR_SOURCE='src'
        SONAR_TOKEN='squ_5790b9342b5d9fae09668b9ed52d4e9170de9088' // Changed from SONAR_LOGIN to SONAR_TOKEN
    }
    stages {
        stage('git_checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/saikrishna0126/spring-framework-petclinic.git'
            }
        }
        // Sonar code quality check
        stage('Sonar Analysis') {
            steps {
                bat 'mvn clean package'
                withSonarQubeEnv(credentialsId: 'sonar-scanner', installationName: 'sonarqube') {
                    bat """
                    %SONAR_SCANNER% ^
                    -Dsonar.projectKey=%SONAR_PROJECTKEY% ^
                    -Dsonar.sources=%SONAR_SOURCE% ^
                    -Dsonar.host.url=%SONAR_URL% ^
                    -Dsonar.login=%SONAR_TOKEN% ^
                    -Dsonar.java.binaries=target/classes 
                    """
                }
            }
        }
        stage('archiveArtifacts') {
            steps {
                archiveArtifacts 'target/*.war'
            }
        }
        stage('tomcat deployment') {
            steps {
                deploy adapters: [Tomcat(url: 'http://34.27.27.61:8080/', credentialsId: 'tomcat')], war: 'target/*.war', contextPath: 'petclinic'
                    
            }
        }
    }
}
