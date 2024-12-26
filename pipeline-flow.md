pipeline {
    agent any

    tools {
        maven "maven3"
    }

    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/caveaku/Master-Github-Action.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=bankapp \
                        -Dsonar.projectName=bankapp \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        stage('Build And Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'devopsshack-settings', maven: 'maven3') {
                    sh "mvn deploy"
                }
            }
        }
        stage('Build And Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t caveaku/bankapp:latest ."
                    }
                }
            }
        }
        stage('Trivy Scan Docker Image') {
            steps {
                sh "trivy image --format table -o image.html caveaku/bankapp:latest"
            }
        }
        stage('Push And Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push caveaku/bankapp:latest"
                    }
                }
            }
        }
        stage('Deploy to K8') {
            steps {
                script {
                   withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://1E3C784699B7E42CE79EC97A01A4755F.gr7.us-east-1.eks.amazonaws.com') {
                    }
                }
            }
        }
    }
