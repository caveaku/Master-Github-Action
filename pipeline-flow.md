pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/caveaku/Master-Github-Action.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
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
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp \
                         -Dsonar.java.binaries=target'''
                }
            }
        }
        stage('Build & Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'devopsshack-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t caveaku/bankapp:latest ."
                    }
                }
            }
        }
        stage('Trivy image Scan') {
            steps {
                sh "trivy image --format table -o image.html cave/bankapp:latest"
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push caveaku/bankapp:latest"
                    }
                }
            }
        }
        stage('Deploy To K8') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://01A78AD7034C792843D102F5E5F10773.gr7.us-east-1.eks.amazonaws.com') {
                    sh "kubectl apply -f ds.yml -n webapps"
                    sleep 30
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://01A78AD7034C792843D102F5E5F10773.gr7.us-east-1.eks.amazonaws.com') {
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}

############################################################################
service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   151m
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl delete all --all
service "kubernetes" deleted
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ ls -al
total 20
drwxrwxr-x 2 ubuntu ubuntu 4096 Dec 28 23:05 .
drwxrwxr-x 4 ubuntu ubuntu 4096 Dec 28 23:04 ..
-rw-rw-r-- 1 ubuntu ubuntu  265 Dec 28 23:05 role-bind.yml
-rw-rw-r-- 1 ubuntu ubuntu  798 Dec 28 23:05 role.yml
-rw-rw-r-- 1 ubuntu ubuntu   83 Dec 28 23:04 svc.yml
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ vi role-bind.yml
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl apply -f .
rolebinding.rbac.authorization.k8s.io/app-rolebinding unchanged
role.rbac.authorization.k8s.io/app-role unchanged
serviceaccount/jenkins unchanged
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ vi secret.yml
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl apply -f secret.yml -n webapps
secret/secname created
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl describe secret secname -n webapps
Name:         secname
Namespace:    webapps
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: jenkins
              kubernetes.io/service-account.uid: 7d584e78-4f39-4c73-b760-8270c7336a14

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1107 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkUwNTdiOXZRd2VrM1RWYVJic0NlQ1JYVWpnT294cUFXUHUtaXBrRERsR3cifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InNlY25hbWUiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiamVua2lucyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjdkNTg0ZTc4LTRmMzktNGM3My1iNzYwLTgyNzBjNzMzNmExNCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDp3ZWJhcHBzOmplbmtpbnMifQ.J5YZxJaiOqsby7-p2cp0_No90ciouUtbxHA7NtbWF1proicqUGayPjVPPf7uc3jHypp5RIUl4fcaOwZVVoK1RmP4WXAfOB5dyD5M1Zn3ockmSAPZrn_t967bcmAd6KRAWEsPVNfL-FTqcRfqX3YNgQrjt0NlhtNFs-cnJidlyxyDJz9NC8AQOzrk-7183_pmqgtJjfhiz6P5dA-5pIQlrvEUqmqsZ6YMtkmC8GpFr9x0rNF-W6h1cCKWdr1N66iIkmZ6j44Jmdbea7P4O1bicQCsiqMPt9GIavM-jwCB46cu8bePV5yguGu4Beqbyy4USzJJ78CWM09Mu21YvSOe1A
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   37m
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP   51m
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl get po
No resources found in default namespace.
ubuntu@ip-172-31-16-44:~/eks/kubernetes$ kubectl get svc -n webapps
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP                                                               PORT(S)        AGE
bankapp-service   LoadBalancer   172.20.1.214   ad2a55ef5be3e4f34b87a9689d01b6f2-1896638230.us-east-1.elb.amazonaws.com   80:32416/TCP   16m
mysql-service     ClusterIP      172.20.85.83   <none>                                                                    3306/TCP       16m
ubuntu@ip-172-31-16-44:~/eks/kubernetes$







