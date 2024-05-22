pipeline{
    agent any
    
    stages{
        stage('Git Checkout'){
            steps{
                git branch: 'main', credentialsId: 'GitHub-HTTPS-Credential', url: 'https://github.com/LalitJ-All-Info/todo.git'
            }
        }
        
        stage('Build Application with Docker'){
            steps{
                sh 'ls -al'
                echo "Building the TODO Application....."
                sh "docker build -t lalitjadaun/todo:${BUILD_TAG} ."
            }
        }
        
        stage('Push to Docker Hub'){
            steps{
                echo "Pushing TODO Application to Dockerhub....."
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUsername')]) {
                    sh "docker login -u ${env.dockerHubUsername} -p ${env.dockerHubPassword}"
                    sh "docker push ${env.dockerHubUsername}/todo:${BUILD_TAG}"
                }
            }
            
        }
        
        stage('Deploy TODO Application')
        {
            when {
                expression {params.Run_Deploy_Stage}
            }
            steps{
                echo "Deploying TODO Application....."
                sh "docker rm -f todo"
                sh "docker run -d --name todo -p 3000:3000 lalitjadaun/todo:${BUILD_TAG}"
            }
        }
    }
}
