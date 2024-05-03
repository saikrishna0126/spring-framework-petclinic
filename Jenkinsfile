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
        SONAR_PROJECTKEY='demo'
        SONAR_SOURCE='src'
        SONAR_TOKEN='squ_5790b9342b5d9fae09668b9ed52d4e9170de9088' // Changed from SONAR_LOGIN to SONAR_TOKEN
        TOMCAT_HOST='34.27.27.61' // IP address or hostname of your remote Tomcat server
        TOMCAT_PORT='8080' // Port on which Tomcat is running
        TOMCAT_USERNAME='tomcat' // Username for accessing Tomcat manager
        TOMCAT_PASSWORD='12345' // Password for accessing Tomcat manager
    }
    
    stages {
        stage('git_checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/saikrishna0126/spring-framework-petclinic.git'
            }
        }
        
        stage('Sonar Analysis') {
            steps {
                bat 'mvn clean package'
                archiveArtifacts 'target/*.war'
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
            
            // Quality Gate check
            stage("Quality Gate") {
                steps {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                script {
                    def buildResult = currentBuild.result
                    if (buildResult == 'SUCCESS') {
                        // Use curl to deploy the WAR file to Tomcat manager
                        bat "curl -v -u ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} --upload-file target/*.war http://${TOMCAT_HOST}:${TOMCAT_PORT}/manager/text/deploy?path=/"
                    } else {
                        echo "Skipping deployment to Tomcat due to build failure"
                    }
                }
            }
        }
    }
}
