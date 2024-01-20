pipeline {
    agent any
    tools{
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/Linux-Genius/DotNet-DEMO.git'
            }
        }
        stage('OWASP Dependency  Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy Fs Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('Soanr Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=dotnet-demo \
                -Dsonar.projectKey=dotnet-demo '''
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "make image"
                    }
                }
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "make push"
                    }
                }
            }
        }
        
        stage('Docker Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker run -d -p 5000:5000 abhisharma1423/dotnet-demoapp"
                    }
                }
            }
        }   
    }
}
