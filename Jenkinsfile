pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PATH = "C:\\Users\\bmd tech\\Documents\\gestion-smartphones\\docker-compose.yml"
        NOTIFY_EMAIL = "daoudaba679@gmail.com"
        SONAR_TOKEN = credentials('sonar_id')
    }

    stages {

        stage('Checkout') {
            steps {
                echo " Clonage du dépôt Git..."
                git branch: 'main', url: 'https://github.com/daouda482/gestion_smarphones.git'
            }
        }

        stage('Install Backend') {
            steps {
                echo " Installation du backend..."
                dir('gestion-smartphone-backend') {
                    bat 'npm install'
                }
            }
        }

        stage('Install & Build Frontend') {
            steps {
                echo " Installation et build du frontend..."
                dir('gestion-smartphone-frontend') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

stage('SonarQube Analysis') {
            steps {
                echo "Analyse du code avec SonarQube"
                withSonarQubeEnv('Sonarqube_local') {
                    withCredentials([string(credentialsId: 'SonarQube_Local', variable: 'SONAR_TOKEN')]) {
                        bat """
                            ${tool('Sonarqube_scanner')}/bin/sonar-scanner \
                            -Dsonar.projectKey=sonarqube \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Docker Build & Up') {
            steps {
                echo " Construction et déploiement des conteneurs Docker..."
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" build"
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" up -d"
            }
        }

        stage('Send Notification') {
            steps {
                echo " Envoi de la notification par mail..."
                mail to: "${NOTIFY_EMAIL}",
                     subject: " Jenkins Build Notification",
                     body: "Le build et le déploiement Jenkins se sont terminés avec succès !"
            }
        }
    }

    post {
        success {
            echo ' Build et déploiement réussis !'
        }
        failure {
            echo ' Le build a échoué.'
        }
    }
}
