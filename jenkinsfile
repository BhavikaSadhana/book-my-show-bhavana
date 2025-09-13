pipeline {
    agent any

    environment {
        REGISTRY = "bhavana686/bookmyshow"
        SONARQUBE = "SonarQube"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'feature', url: 'https://github.com/BhavikaSadhana/book-my-show-bhavana.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }


        stage('Docker Build & Push') {
            steps {
                sh '''
                docker build -t $REGISTRY:v1 .
                docker push $REGISTRY:v1
                '''
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh 'kubectl apply -f k8-manifest/deployment.yaml'
                sh 'kubectl apply -f k8-manifest/service.yaml'
            }
        }

        stage('Notify') {
            steps {
                mail to: 'bhavanasadhana990@gmail.com',
                     subject: "Build #${REGISTRY}",
                     body: "Check Jenkins for details."
            }
        }
    }
}

