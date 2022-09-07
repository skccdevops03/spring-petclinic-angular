pipeline {
    agent none
    
    environment {
            IMAGE_REPO = 'repo-spring-petclinic-angular'
            IMAGE_NAME = 'spring-petclinic-angular'
            IMAGE_TAG = sh(returnStdout: true, script: '(git rev-parse --short HEAD && echo "_$BUILD_NUMBER") | tr -d "\n"').trim()
            REGISTRY_URL = 'http://3.38.12.213:8000'
            REGISTRY_CREDENTIALS = 'credential_harbor'
            ArgoURL='a7660fd42c35b4042a127136f7e294f0-880594388.ap-northeast-2.elb.amazonaws.com/'
            argocdAppPrefix='pet-angular-argocd-helm'
            appWaitTimeout = 60
        }
    
    stages {        
        stage('Build Docker image') {
          agent any
            steps {                  
                script {
                        APP_IMAGE = docker.build("${IMAGE_REPO}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }              
        }
        stage('Push Docker image') {
          agent any
            steps {              
                script {
                    docker.withRegistry(REGISTRY_URL, REGISTRY_CREDENTIALS) {
                    APP_IMAGE.push()
                        APP_IMAGE.push('latest')
                    }
                }                     
            }
        }
        
        stage('Update manifest') {
          agent any    
            steps {
              sh """
                git config --global user.name 'skccdevops03'
                git config --global user.email 'skcc.devops03@sk.com'
                git config --global credential.helper cache
                git config --global push.default simple
              """
              git url: 'https://github.com/skccdevops03/pet-angular-argocd-helm.git', credentialsId: 'credential_git', branch: 'main'
              sh """
                sed -i 's/tag:.*/tag: "${BUILD_NUMBER}"/g' values.yaml
                git add values.yaml
                git commit -m 'Update Docker image tag: ${BUILD_NUMBER}'
                git push origin main
              """
            }
        }    
  
        stage('Argo'){
          agent {
           kubernetes {
             label 'petclinic-cd'
              yamlFile 'jenkins-agent-pod.yaml'
           }
          }
          steps {
          withCredentials([usernamePassword(credentialsId: 'credential_argocd', usernameVariable: 'ARGOCD_USER', passwordVariable: 'ARGOCD_PWD')]) {
            container('argocd') {
                sh """
                yes | argocd login --insecure ${ArgoURL} --username ${ARGOCD_USER} --password ${ARGOCD_PWD}
                argocd app sync ${argocdAppPrefix}
                argocd app wait ${argocdAppPrefix} --timeout ${appWaitTimeout}
                argocd logout ${ArgoURL}
                sleep 10
                """
            }
          }
          }
        }

    }
}