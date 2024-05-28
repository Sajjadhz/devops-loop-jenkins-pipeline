pipeline {
    agent any
    environment {
        DOCKER_REGISTRY_CREDENTIALS = 'docker_creds'
        DOCKER_IMAGE = "sajjadhz/fastapi-app:${env.BUILD_ID}"
        K8S_REPO_URL = 'https://github.com/Sajjadhz/devops-loop-k8s-manifests.git'
        K8S_REPO_CREDENTIALS = 'github_pat'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Sajjadhz/devops-loop-app-scripts.git', credentialsId: 'github_pat'
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
                        sh "sed -i 's|sajjadhz/fastapi-app:.*|${DOCKER_IMAGE}|' deployment.yml"
                        sh 'git config user.email "sajjad.hosseinzadeh@gmail.com"'
                        sh 'git config user.name "sajjadhz"'
                        sh 'git commit -am "Update image tag to ${DOCKER_IMAGE}"'
                        sh 'git push origin main'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(configs: 'k8s/deployment.yml', kubeconfigId: 'devops-loop-kubeconfig')
                }
            }
        }
    }
}
