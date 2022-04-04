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
        stage('Secret scan using TruffleHog') {
            steps {
                sh 'docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm dxa4481/trufflehog file:///src'
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
        stage('Deploy to Kubernetes Dev Environment') {
            steps {
                echo 'Deploy the App using Kubectl'
                sh "sed -i 's/DEPLOYMENTENVIRONMENT/development/g' python-flask-deployment.yml"
                sh "sed -i 's/TAG/$BUILD_NUMBER/g' python-flask-deployment.yml"
                sh "kubectl apply -f python-flask-deployment.yml"
            }
        }
        stage('Promote to Production') {
            steps {
                echo "Promote to production"
            }
            input {
                message "Do you want to Promote the Build to Production"
                ok "Ok"
                submitter "pathinishant@gmail.com"
                submitterParameter "whoIsSubmitter"
                
            }
        }
        stage('Deploy to Kubernetes Production Environment') {
            steps {
                echo 'Deploy the App using Kubectl'
                sh "sed -i 's/development/production/g' python-flask-deployment.yml"
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