pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/aadetunji12/DotNet-DEMO.git'
            }
        }
        
        stage('OWASP Dependency') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Dotnet-demo \
                    -Dsonar.projectKey=Dotnet-demo '''
                }
            }
        } 
        
        stage('Docker build and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        // Change the image tag to 'aadeleke12/dotnet-demoapp:latest' here
                        def dockerImageTag = 'aadeleke12/dotnet-demoapp:latest'
                        sh "docker build . --file build/Dockerfile --tag ${dockerImageTag}"
                    }
                }
            }
        }
        
        stage("Trivy Image Scan") {
            steps {
                sh "trivy image aadeleke12/dotnet-demoapp:latest"
            }
        } 
         
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push aadeleke12/dotnet-demoapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy to container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker run -d -p 5002:50002 aadeleke12/dotnet-demoapp:latest"
                    }
                }
            }
        }
    }
}
