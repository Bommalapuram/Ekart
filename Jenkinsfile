pipeline {
    agent any
    
    tools {
        jdk "jdk11"
        maven "maven"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                // Ee file GitHub lo unte idi auto ga 'scm' nundi checkout chestundi
                checkout scm
            }
        }
        
        stage('Code Compile & Package') {
            steps {
                // JAR file build chesthunnam
                sh "mvn clean package -DskipTests"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=Ekart -Dsonar.projectName=Ekart"
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: '') {
                        // Dockerfile 'docker/' folder lo undi kabatti -f vaduthunnam
                        sh "docker build -f docker/Dockerfile -t devpractice1/ekart:latest ."
                        sh "docker push devpractice1/ekart:latest"
                    }
                }
            }
        }

        stage('Docker Deploy') {
            steps {
                script {
                    // Patha container unte stop chesi remove chesthundhi
                    sh "docker stop ekart-container || true"
                    sh "docker rm ekart-container || true"

                    // Kotha image ni run chesthundhi
                    sh "docker run -d --name ekart-container -p 8070:8070 devpractice1/ekart:latest"
                }
            }
        }
    }

    post {
        always {
            echo "Build Process Completed."
        }
        success {
            echo "Successfully Deployed! Access at http://your-server-ip:8070"
        }
    }
}
