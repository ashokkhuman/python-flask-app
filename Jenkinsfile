pipeline { 
    environment { 
        registry = "ashokdocker123/python-flask" 
        registryCredential = 'ashokDockerCredential' 
        dockerImage = '' 
    }
    agent any 
    stages { 
        stage('Cloning our Git') { 
            steps { 
                git branch:'main', url: 'https://github.com/ashokkhuman/python-flask-app.git' 
            }
        } 
        stage('Security scan') {
            steps {
                sh 'docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm secfigo/bandit bandit -r /src -f json -o /src/bandit-output.json | exit 0'
            }
        }
        stage('Building our image') { 
            steps { 
                script { 
                    dockerImage = docker.build registry + ":$BUILD_NUMBER" 
                }
            }
        } 
        stage('Deploy our image') { 
            steps { 
                script { 
                    docker.withRegistry( '', registryCredential ) { 
                        dockerImage.push() 
                    }
                } 
            }
        } 
        stage('Cleaning up') { 
            steps { 
                sh "docker rmi $registry:$BUILD_NUMBER" 
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploy the App using Kubectl'
                sh "sed -i 's/TAG/$BUILD_NUMBER/g' python-flask-deployment.yml"
                sh "kubectl apply -f python-flask-deployment.yml"
            }
        }
         
    }
    post {
        always {
            archiveArtifacts artifacts: 'bandit-output.json',onlyIfSuccessful: true
        }
    }
}