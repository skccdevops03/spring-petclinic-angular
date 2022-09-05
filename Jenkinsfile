pipeline {
    agent any
    
    environment {
            GIT_COMMIT_SHORT = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            APP_IMAGE = null
            IMAGE_REPO = 'repo-spring-petclinic-angular'
            IMAGE_NAME = 'spring-petclinic-angular'
            //IMAGE_TAG = '${BUILD_ID}_${BUILD_NUMBER}'
            IMAGE_TAG = sh(returnStdout: true, script: '(git rev-parse --short HEAD && echo "_$BUILD_NUMBER") | tr -d "\n"').trim()
            REGISTRY_URL = 'http://3.38.12.213:8000'
            REGISTRY_CREDENTIALS = 'credential_harbor'
        }
    
    stages {
        
        stage('Build Docker image') {
            steps {
                   
                script {
                        APP_IMAGE = docker.build("${IMAGE_REPO}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }   
              
            }
        }
        stage('Push Docker image') {
            steps {
              
                script {
                    docker.withRegistry(REGISTRY_URL, REGISTRY_CREDENTIALS) {
                    APP_IMAGE.push()
                        APP_IMAGE.push('latest')
                    }
                }
                      
            }
        }
        
    }    
}