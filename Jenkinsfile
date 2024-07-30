pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS' // Ensure the name matches the NodeJS installation configured in Jenkins
    }
    
    environment {
        FRONTEND_REPO = 'https://github.com/Sujithsai08/carshowroom_frontend.git'
        FRONTEND_BRANCH = 'main'
       
        REGISTRY_CREDENTIALS = credentials('docker-cred')
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: "${env.FRONTEND_REPO}", branch: "${env.FRONTEND_BRANCH}"
            }
        }
        
        stage('Install Dependencies') {
            steps {
                withEnv(['CI=false']) {
                    sh 'npm install'
                }
            }
        }
        
        stage('Build Application') {
            steps {
                sh 'npm run build' 
            }
        }
        
        
        
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "sujithsai/carshowroom:${env.BUILD_NUMBER}"
            }
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "carshowroom_frontend"
                GIT_USER_NAME = "Sujithsai08"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                    git config user.email "sujithsai.sirimalla33@gmail.com"
                    git config user.name "${GIT_USER_NAME}"
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifests/deployment.yaml
                    git add manifests/deployment.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GIT_USER_NAME}:${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Clean up any workspace or resources
            cleanWs()
        }
    }
}
