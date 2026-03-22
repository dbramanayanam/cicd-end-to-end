pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'github_credentials', 
                url: 'https://github.com/dbramanayanam/cicd-end-to-end.git',
                branch: 'main'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    sh '''
                    echo 'Buid Docker Image'
                    docker build -t dbramayanam/cicd-e2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

       stage('Push the artifacts'){
           steps{
              script{
                 withCredentials([
                 usernamePassword(
                    credentialsId: 'dockerhub_credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                  )
                ]) {
                  sh '''
                   echo $DOCKER_PASS | docker login \
                   -u $DOCKER_USER \
                   --password-stdin
                  
                docker push dbramayanam/cicd-e2e:${BUILD_NUMBER}
                
                docker logout
                '''
              }
          }
      }
  }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'github_credentials', 
                url: 'https://github.com/dbramanayanam/cicd-demo-manifests-repo.git',
                branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'github_credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy.yaml
                        sed -i "s|dbramayanam/cicd-e2e:.*|dbramayanam/cicd-e2e:${BUILD_NUMBER}|g" deploy.yaml
                        cat deploy.yaml
                        git config user.email "dbramanayanam@gmail.com"
                        git config user.name "dbramanayanam"
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/dbramanayanam/cicd-demo-manifests-repo.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
