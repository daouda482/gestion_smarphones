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
                stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyse du code avec SonarQube..."
                withSonarQubeEnv('SonarQube_Local') {  // nom du serveur SonarQube défini dans Jenkins
                    withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_TOKEN')]) {
                        sh """
                            sonar-scanner \
                              -Dsonar.projectKey=fil-rouge \
                              -Dsonar.projectName='Projet Fil Rouge' \
                              -Dsonar.projectVersion=1.0 \
                              -Dsonar.sources=. \
                              -Dsonar.exclusions=/node_modules/,/build/,/dist/,/.test.js,/.spec.js \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.token=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }


        stage('Docker Build & Up') {
    steps {
        bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" build"
        bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" up -d"
    }
}

         stage('Send Notification') {
            steps {
                 mail to: "${NOTIFY_EMAIL}",
                     subject: "Jenkins Build Notification",
                     body: "The Jenkins build and deployment process has completed successfully."
            }
         }
    }

    post {
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build failed.'
        }
    }

}
