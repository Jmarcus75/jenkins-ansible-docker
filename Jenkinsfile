node {
    // Variables
    def registryProjet = 'registry.gitlab.com/xavki/presentations-jenkins/wartest'
    def IMAGE = "${registryProjet}:version-${env.BUILD_ID}"

    // Stage 1: Cloner le projet Maven
    stage('Build - Clone') {
        git 'https://github.com/priximmo/war-build-docker.git'
    }

    // Stage 2: Maven package
    stage('Build - Maven package') {
        sh 'mvn package'
    }

    // Stage 3: Build Docker image
    def img = docker.build("$IMAGE", '.')
    
    // Stage 4: Test de l'image Docker
    stage('Build - Test') {
        img.withRun("--name run-${env.BUILD_ID} -p 8081:8080") { c ->
            sh 'docker ps'
            sh 'netstat -ntaup || true'   // Permet d'éviter une erreur si netstat est limité
            sh 'sleep 10s'                // Laisser Tomcat démarrer
            sh 'curl -f http://127.0.0.1:8081'  // Vérifie que l'application répond
            sh 'docker ps'
        }
    }

    // Stage 5: Push Docker image sur GitLab
    stage('Build - Push') {
        docker.withRegistry('https://registry.gitlab.com', 'reg1') {
            img.push('latest')  // push latest
            img.push()          // push version spécifique
        }
    }

    // Stage 6: Cloner le repo Ansible pour le déploiement
    stage('Deploy - Clone') {
        git 'https://github.com/priximmo/jenkins-ansible-docker.git'
    }

    // Stage 7: Déploiement avec Ansible sur la VM cible
    stage('Deploy - End') {
        ansiblePlaybook(
            colorized: true,
            become: true,
            playbook: 'playbook.yml',
            inventory: "${env.HOST},",      // <-- prend la valeur du paramètre Jenkins
            extras: "--extra-vars 'image=$IMAGE'"
        )
    }
}
