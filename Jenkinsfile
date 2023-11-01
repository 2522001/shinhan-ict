node {
    stage('Clone repository') {
        checkout scm
    }
    stage('Build image') {
        app = docker.build("minseo205/test_pipeline")
    }
    stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker_credential') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
}