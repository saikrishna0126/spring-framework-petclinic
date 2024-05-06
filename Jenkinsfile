pipeline {
    agent any
    
    tools {
        maven 'maven' // Make sure Maven tool is configured in Jenkins
        // You can define other tools here as needed
    }
    
    environment {
        SONAR_SCANNER = 'C:\\Sonarscanner\\sonar-scanner-5.0.1.3006-windows\\bin\\sonar-scanner.bat'
    }
    
    stages {
        stage('git_checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/saikrishna0126/spring-framework-petclinic.git'
            }
        }
        
        stage('Sonar Analysis and Deploy to Tomcat') {
            steps {
                // Sonar code quality check
                bat 'mvn clean package'
                
                // Archive artifacts
                archiveArtifacts 'target/*.war'
                
                // Sonar analysis
                withSonarQubeEnv(credentialsId: 'sonar-scanner', installationName: 'sonarqube') {
                    bat """
                     %SONAR_SCANNER% ^
                    -Dsonar.projectKey="${env.SONAR_PROJECT_KEY}" ^
                    -Dsonar.sources=src ^
                    -Dsonar.host.url="${env.SONAR_SERVER_URL}" ^
                    -Dsonar.login="${env.SONAR_TOKEN}" ^
                    -Dsonar.java.binaries=target/classes 
                    """
                }
                
                // Quality Gate check
                timeout(time: 1, unit: 'HOURS') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                        else {
                            print "Pipeline Executed successfully: ${qg.status}"
                            
                            // Deploy to Tomcat if quality gate passes
                            deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: "${env.TOMCAT9_URL}")], contextPath: null, war: '**/*.war'
                        }
                    }
                }
            }
        }
    }
}
