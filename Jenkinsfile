pipeline {
  agent any
  environment {
    dockerHubRegistry = 'minseo205/test_pipeline'
    dockerHubRegistryCredential = 'docker_credentials'
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
	    sh "git config --global user.name '2522001'"
	    sh "git config --global user.email 'minseo770@gmail.com'"
            sh "git checkout -B main"

            git credentialsId: 'github_signin',
                url: 'https://github.com/2522001/k8s-manifest.git',
                branch: 'main'

            sh "sed -i 's/test:.*\$/test:${currentBuild.number}/g' deployment.yaml"
            sh "git add deployment.yaml"
            sh "git commit -m '[UPDATE] test ${currentBuild.number} image versioning'"
            sh "git remote set-url origin git@github.com:2522001/k8s-manifest.git"
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