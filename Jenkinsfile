pipeline {
    agent { label 'docker' }

    environment {
        DOCKER_IMAGE = 'devopsfarm/todo'
        DOCKER_CREDENTIALS_ID = 'dockerHub'
        GIT_CREDENTIALS_ID = 'GitHub-HTTPS-Credential'
        KUBECONFIG_CREDENTIALS_ID = 'Kubeconfig-Credential'
        GIT_BRANCH = 'main'
    }

    parameters {
        booleanParam(name: 'Run_Deploy_Stage', defaultValue: true, description: 'Run deployment stage')
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm: [
                    $class: 'GitSCM',
                    branches: [[name: "${GIT_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/LalitJ-All-Info/todo.git',
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ]
            }
        }

        stage('Build Application with Docker') {
            steps {
                script {
                    echo "Building the TODO Application....."
                    sh "docker build -t todo_app ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing TODO Application to Dockerhub....."
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUsername')]) {
                        sh "docker login -u ${dockerHubUsername} -p ${dockerHubPassword}"
                        sh "docker tag todo_app ${DOCKER_IMAGE}:${BUILD_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_TAG}"
                    }
                }
            }
        }

        stage('Deploy TODO Application') {
            when {
                expression { params.Run_Deploy_Stage }
            }
            steps {
                script {
                    echo "Deploying TODO Application to Kubernetes....."
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        sh """
                        kubectl apply -f webapp-deployment.yaml
                        kubectl apply -f webapp-service.yaml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}
