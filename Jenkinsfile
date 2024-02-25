pipeline {
    agent any
    tools { 
        maven 'Maven_3_5_2'  
    }
    stages {
        stage('CompileandRunSonarAnalysis') {
            steps {	
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=pramoth-devops28_sast -Dsonar.organization=pramoth-devops28 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=e57ee3e6a5cbb809f831a7cd83b4a9c7561a6250'
            }
        }

        stage('RunSCAAnalysisUsingSnyk') {
            steps {		
                echo 'Testing...'
                snykSecurity(
                    snykInstallation: 'synktool',
                    snykTokenId: 'synk_token'
                )
            }
        }

        stage('Build') { 
            steps { 
                withDockerRegistry([credentialsId: "docker", url: ""]) {
                    script {
                        app = docker.build("asg")
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://056296809003.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:aws-cred') {
                        app.push("latest")
                    }
                }
            }
        }

        stage('Kubernetes Deployment of ASG Bugg Web Application') {
            steps {
                withKubeConfig([credentialsId: 'kubelogin']) {
                    sh('kubectl delete all --all -n devsecops')
                    sh('kubectl apply -f deployment.yaml --namespace=devsecops')
                }
            }
        }

        stage('wait_for_testing') {
            steps {
                sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
            }
        }
        
        stage('RunDASTUsingZAP') {
            steps {
                withKubeConfig([credentialsId: 'kubelogin']) {
                    sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
                    archiveArtifacts artifacts: 'zap_report.html'
                }
            }
        } 
    }
}
