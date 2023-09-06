pipeline {
    agent any 
    
    tools{
        jdk 'jdk-17.0.8.1'
        maven 'maven 3.9.4'
    }
    
    environment {
        SCANNER_HOME=tool 'scanner-sonar'
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/writetoritika/Petclinic.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'ace15a9f-7a24-4292-91ee-1bd2c9c84abf', toolName: 'docker') {
                        
                        sh "docker build -t petclinic1 ."
                        sh "docker tag petclinic1 andreibelyi/pet-clinic123:latest "
                        sh "docker push andreibelyi/pet-clinic123:latest "
                    
                    }
                }
            }
        }
        stage("Trivy") {
            steps{
                sh 'trivy --no-progress --exit-code 1 --severity HIGH,CRITICAL andreibelyi/pet-clinic123:latest'
            }
        }
        stage("Deploy Using Docker"){
            steps{
                sh " docker run -d --name pet1 -p 8001:8082 andreibelyi/pet-clinic123:latest "
            }
        }
        stage("Deploy To Tomcat"){
            steps{
                sh "cp  target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }
     }
}
