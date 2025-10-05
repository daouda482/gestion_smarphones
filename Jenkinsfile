pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PATH = "C:\\Users\\bmd tech\\Documents\\gestion-smartphones\\docker-compose.yml"
        NOTIFY_EMAIL = "daoudaba679@gmail.com"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/daouda482/gestion_smarphones.git'
            }
        }

        stage('Install Backend') {
            steps {
                dir('gestion-smartphone-backend') {
                    bat 'npm install'
                }
            }
        }

        stage('Install & Build Frontend') {
            steps {
                dir('gestion-smartphone-frontend') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Docker Build & Up') {
    steps {
        bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" build"
        bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" up -d"
    }
}
    }

   post {
        success {
            echo '✅ Build réussi.'
            mail to: "${NOTIFY_EMAIL}",
                 subject: "✅ Build réussi : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Le build s'est terminé avec succès.\n\nDétails : ${env.BUILD_URL}"
        }

        failure {
            echo '❌ Build échoué.'
            mail to: "${NOTIFY_EMAIL}",
                 subject: "❌ Build échoué : ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Le build a échoué.\n\nConsultez les logs ici : ${env.BUILD_URL}"
        }
    }

}
