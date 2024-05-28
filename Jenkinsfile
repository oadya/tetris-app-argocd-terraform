pipeline {

    agent any
    
    environment {
      SCANNER_HOME=tool 'sonar-scanner'
      NAME = "tetrisv1"
      VERSION = "${env.BUILD_ID}-${env.GIT_COMMIT}"
      IMAGE_REPO = "orpheadya"
      GITHUB_TOKEN = credentials('github-token')
      NEW_IMAGE_NAME = ''
      GIT_MANIFEST_REPO_NAME = "Tetris-manifest"
      GIT_USER_NAME = "oadya"
    }
    
    
   tools {
       jdk 'jdk17'
       nodejs 'node16'
    }
    
    stages {
        
        stage('Cloning the project') {
            steps {
                git branch: 'main', url: 'https://github.com/oadya/tetris-app-argocd-terraform.git'
            }
        }
        
        stage('NPM') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('TRIVY FS') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        
        stage("Build and Push") {
            steps {
                script{
                    
                    // This step should not normally be used in your script. Consult the inline help for details.
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        
                      sh "docker build -t ${NAME} ."
                      sh "docker tag ${NAME}:latest ${IMAGE_REPO}/${NAME}:${VERSION}"
                      sh 'docker push ${IMAGE_REPO}/${NAME}:${VERSION}'
                    }
                }
            }
        }
        
        stage('TRIVY Image') {
            steps {
                sh 'trivy image orpheadya/tetrisv1:latest > trivyImage.txt'
            }
        }
        
        // Update kubernetes manifest deployments  with the new version of the image
        stage("Clone or Pull Manifest Repo") {
            steps {

               script {

                  if(fileExists('Tetris-manifest')) {

                    echo 'Cloned repo already exists - Pulling the latest changes'

                    dir('Tetris-manifest') {
                        sh 'git pull'
                    }

                  } else {

                    echo 'Repo does not exists - Cloning the repo'
                    sh 'git clone -b main https://github.com/oadya/Tetris-manifest.git'

                  }
               }
            }
        }

        stage('Update manifest') {
            steps {
                script {
                    
                    NEW_IMAGE_NAME = "${IMAGE_REPO}/${NAME}:${VERSION}"
                    dir('Tetris-manifest') {
                        sh "sed -i 's|image: .*|image: $NEW_IMAGE_NAME|' deployment.yml"
                        sh 'cat deployment.yml'
                    }
              }
              
            }
     
        }

        stage('Commit & Push') {
            steps {
                dir('Tetris-manifest') {
                    sh "git remote set-url origin https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_MANIFEST_REPO_NAME}"
                    sh 'git checkout main'
                    sh 'git add -A'
                    sh 'git commit -am "Updated image version for build - $VERSION"'
                    sh 'git push origin main'
                }
            }
     
        }
        
    }
    
}
