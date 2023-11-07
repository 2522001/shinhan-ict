pipeline {
  agent any
  environment {
    dockerHubRegistry = 'minseo205/k8s-project'
    dockerHubRegistryCredential = 'docker-credentials'
    githubCredential = 'github-credentials'
    gitEmail = 'minseo770@gmail.com'
    gitName = '2522001'
  }

  stages {

    stage('Checkout Application Git Branch') {
        steps {
            checkout scm
        }
        post {
                failure {
                  echo 'Repository clone failure !'
                }
                success {
                  echo 'Repository clone success !'
                }
        }
    }

    stage('Docker Image Build') {
        steps {
            sh "docker build . -t ${dockerHubRegistry}:${currentBuild.number}"
            sh "docker build . -t ${dockerHubRegistry}:latest"
        }
        post {
                failure {
                  echo 'Docker image build failure !'
                }
                success {
                  echo 'Docker image build success !'
                }
        }
    }

    stage('Docker Image Push') {
        steps {
            withDockerRegistry([ credentialsId: dockerHubRegistryCredential, url: "" ]) {
                                sh "docker push ${dockerHubRegistry}:${currentBuild.number}"
                                sh "docker push ${dockerHubRegistry}:latest"

                                sleep 10 /* Wait uploading */ 
                            }
        }
        post {
                failure {
                  echo 'Docker Image Push failure !'
                  sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
                  sh "docker rmi ${dockerHubRegistry}:latest"
                }
                success {
                  echo 'Docker image push success !'
                  sh "docker rmi ${dockerHubRegistry}:${currentBuild.number}"
                  sh "docker rmi ${dockerHubRegistry}:latest"
                }
        }
    }

    stage('K8S Manifest Update') {
        steps {
            git credentialsId: githubCredential,
                url: 'https://github.com/2522001/test.git',
                branch: 'main'

	    sh "git config --global user.email ${gitEmail}"
            sh "git config --global user.name ${gitName}"
            sh "sed -i 's/k8s-project.*\$/k8s-project:${currentBuild.number}/g' deployment.yaml"
            sh "git add ."
            sh "git commit -m 'fix:${dockerHubRegistry} ${currentBuild.number} image versioning'"
            sh "git branch -M main"
            sh "git remote remove origin"
            sh "git remote add origin https://github.com/2522001/test.git"
            sh "git push -u origin main"
        }
        post {
                failure {
                  echo 'K8S Manifest Update failure !'
                }
                success {
                  echo 'K8S Manifest Update success !'
                }
        }
    }
    
  }
}