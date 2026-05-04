pipeline {
    agent any
    
    tools {
        jdk "jdk11"
        maven "maven"
    }
    
    stages {
        // --- SECTION 1: CONTINUOUS INTEGRATION (CI) ---
        
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Bommalapuram/Ekart.git'
            }
        }
        
        stage('Code Compile & Package') {
            steps {
                // 'package' JAR file ni create chestundi (Docker build ki idi avasaram)
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

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: '') {
                        // Dockerfile folder lo unna path ni -f tho correct ga point chesam
                        sh "docker build -f docker/Dockerfile -t devpractice1/ekart:latest ."
                        sh "docker push devpractice1/ekart:latest"
                    }
                }
            }
        }

        // --- SECTION 2: CONTINUOUS DEPLOYMENT (CD) ---

        stage('Docker Deploy to Container') {
            steps {
                script {
                    // Pata container unte stop chesi remove chestundi, lekapothe skip avthundi (|| true)
                    sh "docker stop ekart-container || true"
                    sh "docker rm ekart-container || true"

                    // Kotha image ni 8070 port meeda run chestundi
                    sh "docker run -d --name ekart-container -p 8070:8070 devpractice1/ekart:latest"
                }
            }
        }
    }

    // Future-proofing: Build ayyaka workspace clean cheyyadaniki
    post {
        always {
            echo "Pipeline execution finished."
        }
        success {
            echo "Deployment successful! Check the application at http://your-ip:8070"
        }
        failure {
            echo "Pipeline failed. Check logs for more details."
        }
    }

            stage('Kubernetes Deploy') {
            steps {
                script {
                    // Kubernetes config file ni use cheసి deployment చేస్తాం
                    // 'k8s-config' అనేది Jenkins Credentials లో మీరు క్రియేట్ చేయాల్సిన ID
                    withKubeConfig([credentialsId: 'k8s-config']) {
                        // 1. Deployment and Service files ని అప్లై చేయడం
                        sh "kubectl apply -f k8s/deployment.yaml"
                        sh "kubectl apply -f k8s/service.yaml"
                        
                        // 2. Rolling update ని ఫోర్స్ చేయడం (కొత్త ఇమేజ్ ని పుల్ చేయడానికి)
                        sh "kubectl rollout restart deployment ekart-deployment"
                    }
                }
            }
        }

}
