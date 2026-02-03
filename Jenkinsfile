node {
    // Variables pour Docker et GitLab
    def registryProjet = 'registry.gitlab.com/jmarcus75/presentations-jenkins20/wartest'
    def IMAGE = "${registryProjet}:version-${env.BUILD_ID}"

    stage('Build - Clone Maven project') {
        git 'https://github.com/priximmo/war-build-docker.git'
    }

    stage('Build - Maven package') {
        sh 'mvn package'
    }

    stage('Build - Docker image') {
        // Build Docker image
        def img = docker.build("$IMAGE", '.')
        // Stocker l'image pour les étapes suivantes
        env.DOCKER_IMAGE = IMAGE
    }

    stage('Build - Test Docker image') {
        def img = docker.image(env.DOCKER_IMAGE)
        img.withRun("--name run-${env.BUILD_ID} -p 8081:8080") { c ->
            sh 'docker ps'
            sh 'sleep 10s'
            sh 'curl -f http://127.0.0.1:8081'
            sh 'docker ps'
        }
    }

    stage('Build - Push Docker image') {
        // Push Docker image sur GitLab en utilisant les credentials Jenkins
        withCredentials([string(credentialsId: 'reg1', variable: 'GITLAB_TOKEN')]) {
            docker.withRegistry('https://registry.gitlab.com', '') {
                sh """
                echo "$GITLAB_TOKEN" | docker login registry.gitlab.com -u Jmarcus75 --password-stdin
                docker push $IMAGE
                docker push ${IMAGE}:latest
                """
            }
        }
    }

    stage('Deploy - Clone Ansible repo') {
        git 'https://github.com/Jmarcus75/jenkins-ansible-docker.git'
    }

    stage('Deploy - Run Ansible playbook') {
        withCredentials([string(credentialsId: 'reg1', variable: 'GITLAB_TOKEN')]) {
            ansiblePlaybook(
                colorized: true,
                become: true,
                playbook: 'playbook.yml',
                inventory: "${env.HOST},",
                extras: "--extra-vars 'image=$IMAGE gitlab_token=$GITLAB_TOKEN'"
            )
        }
    }
}
