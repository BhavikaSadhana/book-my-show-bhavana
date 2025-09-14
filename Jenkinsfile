pipeline {
    agent any

    tools {
        jdk 'jdk17'
        node 'node24'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'bhavana686/bookmyshow:v4'
        K8S_NAMESPACE = 'bhavana'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git branch: 'feature', url: 'https://github.com/BhavikaSadhana/book-my-show-bhavana.git'
                sh 'ls -la'  // Verify files after checkout
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                        $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectName=BookMyShow \
                            -Dsonar.projectKey=BMS
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    cd bookmyshow-app
                    ls -la  # Verify package.json exists
                    if [ -f package.json ]; then
                        rm -rf node_modules package-lock.json  # Remove old dependencies
                        npm install  # Install fresh dependencies
                    else
                        echo "Error: package.json not found in bookmyshow-app!"
                        exit 1
                    fi
                '''
            }
        }

        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit',
                                odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh ''' 
                            echo "Building Docker image..."
                            docker build --no-cache -t $DOCKER_IMAGE -f bookmyshow-app/Dockerfile bookmyshow-app

                            echo "Pushing Docker image to registry..."
                            docker push $DOCKER_IMAGE
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying to Kubernetes namespace: ${K8S_NAMESPACE}"

                    kubectl apply -f k8-manifest/deployment.yaml
                    kubectl apply -f k8-manifest/service.yaml

                    echo "Verifying deployment..."
                    kubectl get pods -n ${K8S_NAMESPACE}
                    kubectl get svc -n ${K8S_NAMESPACE}
                '''
            }
        }
    }

    post {
        always {
            emailext(
                attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                      "Build Number: ${env.BUILD_NUMBER}<br/>" +
                      "URL: ${env.BUILD_URL}<br/>",
                to: 'bhavanasadhana01@gmail.com',
                attachmentsPattern: 'trivyfs.txt'
            )
        }
    }
}
