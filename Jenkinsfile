pipeline {
    agent any
    environment {
        DOCKER_REGISTRY_CREDENTIALS = 'docker-registry-credentials-id'
        DOCKER_IMAGE = "your-docker-registry/fastapi-app:${env.BUILD_ID}"
        K8S_REPO_URL = 'https://github.com/your-username/repo-b.git'
        K8S_REPO_CREDENTIALS = 'your-k8s-repo-credentials-id'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/your-username/repo-a.git', credentialsId: 'your-github-credentials-id'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_REGISTRY_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    dir('k8s') {
                        git url: K8S_REPO_URL, credentialsId: K8S_REPO_CREDENTIALS
                        sh "sed -i 's|your-docker-registry/fastapi-app:.*|${DOCKER_IMAGE}|' k8s-deployment.yml"
                        sh 'git config user.email "jenkins@yourdomain.com"'
                        sh 'git config user.name "jenkins"'
                        sh 'git commit -am "Update image tag to ${DOCKER_IMAGE}"'
                        sh 'git push origin main'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(configs: 'k8s/k8s-deployment.yml', kubeconfigId: 'kubeconfig-credentials-id')
                }
            }
        }
    }
}
