pipeline { 
    agent {
        label 'qa'
    }
    tools {
        maven 'maven3'
    }
    environment {
        scanner = tool 'sonarscanner'
        userName = 'deepthinker07'
        imgName = 'bgimg_qa'
    }
    stages {
        stage ('cleanWs') {
            steps {
                cleanWs()
            }
        }
        stage ('git checkout') {
            steps {
                git credentialsId: 'sa', url: 'https://github.com/Deepthinker07/multi-stage.git', branch: 'qa'
            }
        }    
        stage ('compile') {
            steps {
                sh 'mvn compile'
                    } 
                }      
        stage ('unit test') {
            steps {
                sh 'mvn test'
                    }
                } 
        stage ('static code analysis') {
            steps {
                withSonarQubeEnv ('sonarserver') {
                    sh '''
                    $scanner/bin/sonar-scanner \
                    -Dsonar.projectName=boardgame-qa \
                    -Dsonar.projectKey=boardgamekey \
                    -Dsonar.java.binaries=target/classes 
                    '''
                }
            }
        }
        stage ('code package') {
            steps {
                sh 'mvn package'
            }
        }
        stage ('docker image build') {
            steps {
                sh 'docker build -t $userName/$imgName .'
            }
        }
        stage ('push image to dockerhub') {
            steps {
                withDockerRegistry(credentialsId: 'sa2', url: '') {
                    sh 'docker push $userName/$imgName'
                }    
            }
        }
        stage ('deploy to eks') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'singh-EKS', contextName: '', credentialsId: 'qa-token-k8', namespace: 'qa', restrictKubeConfigAccess: false, serverUrl: 'https://D0684F104A0902FC351413D1CAE89130.gr7.ap-southeast-1.eks.amazonaws.com') {
                    sh 'cd /home/ubuntu/qa/workspace/bg-qa'
                    sh 'kubectl apply -f deploy-svc.yml'
                    sleep 60
                }
            }
        }
        stage ('verify to eks') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'singh-EKS', contextName: '', credentialsId: 'qa-token-k8', namespace: 'qa', restrictKubeConfigAccess: false, serverUrl: 'https://D0684F104A0902FC351413D1CAE89130.gr7.ap-southeast-1.eks.amazonaws.com') {
                    sh 'kubectl get svc'
                }
            }
        }
            
    }  
}