pipeline {
    agent any
   
    tools{
        maven "maven3"
        jdk "jdk17"
    }
   
   environment {
     SCANNER_HOME= tool "sonar-scanner"
     }
    stages {
        stage('GIT Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/manishaacharya111/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART \
                    -Dsonar.projectName=EKART \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
                
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                  sh "mvn deploy -DskipTests=true"
               }
            }
        }
        
        stage('Build & tag Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                        sh "docker build -t merry211/ekart:latest -f docker/Dockerfile ."
                    } 
                }
                
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy image merry211/ekart:latest > trivy-report.txt"
            }
        }
        
        stage('Push the docker image to DockerHub') {
          steps {
              script{
                 withDockerRegistry(credentialsId: 'docker-credential', toolName: 'docker') {
                     sh "docker push merry211/ekart:latest"
                     }
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.61.203:6443') {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}

