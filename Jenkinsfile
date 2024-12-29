pipeline {
    agent any
    
    tools {
        maven 'maven'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = 'sriram8788/bankapp'
        TAG = "${env.BUILD_NUMBER}"
        
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/praviteja999/Fullstack-MultiTier-Java.git'

            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('FS Scan - Trivy') {
            steps {
                sh "trivy fs --format table -o fs.html ."

            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=multitier3 -Dsonar.projectName=multitier -Dsonar.java.binaries=target"
                }

            }
        }
        
        
        /*stage('Quality GateCheck') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
            }
        }*/
        
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        
        stage('Publish Artifact To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                    
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'dockerhub-token') {
                    sh 'docker system prune -f'
                    sh 'docker container prune -f'
                    sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    
                    }

                }
            }
        }
        
        stage('Image Scan - Trivy') {
            steps {
                sh "trivy image --format table -o image.html ${IMAGE_NAME}:${TAG}"

            }
        }
        
        stage('Push Docker Image') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'dockerhub-token') {
                    sh "docker push ${IMAGE_NAME}:${TAG}"
                    
                    }

                }
            }
        }
        
        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    // Replace the Docker image tag in line 58 of the ds.yml file
                    sh """
                    sed -i '58s|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${TAG}|' ds.yml
                    """
                }
            }
        }
        
        stage('Commit and Push Changes') {
            steps {
                script {
                    // Use GitHub credentials for Jenkins
                withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                sh """
                git config --global user.email "praviteja999@protonmail.com"
                git config --global user.name "praviteja999"
                git remote set-url origin https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/praviteja999/Fullstack-MultiTier-Java.git
                git pull origin main
                git add .
                git commit -m "Update image to ${IMAGE_NAME}:${TAG}"
                git push origin main
                """
                    }
                }
            }
        }
        
        
        /*stage('Kubernetes Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devops-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://E1F67E40D41E01B2E0EE8B5DEFBE9E90.gr7.ap-south-1.eks.amazonaws.com') {
                sh "kubectl apply -f ds.yml"
                sleep 30
                }
            }
        }
        
        stage('Kubernetes Deployment Verification') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devops-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://E1F67E40D41E01B2E0EE8B5DEFBE9E90.gr7.ap-south-1.eks.amazonaws.com') {
                sh "kubectl get pods -n webapps"
                sh "kubectl get svc -n webapps"
                }
            }
        }*/

    }
}
