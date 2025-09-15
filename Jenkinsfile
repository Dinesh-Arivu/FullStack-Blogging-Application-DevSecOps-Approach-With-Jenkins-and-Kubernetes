pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/Dinesh-Arivu/FullStack-Blogging-Application-DevSecOps-Approach-With-Jenkins-and-Kubernetes.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Trivy File Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=FullstackBlog -Dsonar.projectKey=FullstackBlog \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish To Nexus') {
            steps {
            withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                            sh "docker build -t dinesh1097/fullstackblog:latest ."
                            sh "docker tag fullstackblog dinesh1097/fullstackblog:latest "
                    }
               }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html dinesh1097/fullstackblog:latest "
            }
        }
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                            sh "docker push dinesh1097/fullstackblog:latest"
                    }
               }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                      sh "kubectl apply -f K8S/deployment.yaml"
                }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh "kubectl get pods "
                        sh "kubectl get svc "
                }
            }
        }
    }   
}
