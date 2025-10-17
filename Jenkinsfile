pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PATH = "C:\\Users\\bmd tech\\Documents\\gestion-smartphones\\docker-compose.yml"
        NOTIFY_EMAIL = "daoudaba679@gmail.com"
 
        SONAR_TOKEN = credentials('sonar_db')

        SONARQUBE_ENV = 'SonarQubeServer' // Nom configuré dans Jenkins
        SCANNER_TOOL = 'SonarQube_Scanner' // Nom du scanner ajouté dans Global Tool Configuration
  (J'ai modifié mon fichier jenkinsfile à nouveau)
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
                withSonarQubeEnv('SonarQube_Local') {
                    withCredentials([string(credentialsId: 'sonar_db', variable: 'SONAR_TOKEN')]) {
                        bat """
                            ${tool('SonarQube_Scanner')}/bin/sonar-scanner \
                            -Dsonar.projectKey=sonarqube \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=$SONAR_TOKEN
                        """

                script {
                    // Injection de l'environnement SonarQube configuré dans Jenkins
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        // Détection automatique du chemin du sonar-scanner
                        def scannerHome = tool name: "${SCANNER_TOOL}", type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                        bat """
                            "${scannerHome}\\bin\\sonar-scanner" ^
                            -Dsonar.projectKey=gestion-smartphones ^
                            -Dsonar.projectName="Gestion Smartphones" ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=${SONAR_HOST_URL} ^
                            -Dsonar.login=${SONAR_AUTH_TOKEN}
                        """
                    }
                }
            }
        }

        // ✅ Vérification du Quality Gate
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        echo "Quality Gate status: ${qg.status}"
                        if (qg.status != 'OK') {
                            error "❌ Build stopped — Quality Gate failed (${qg.status})"
                        } else {
                            echo "✅ Quality Gate passed!"
                        }
(J'ai modifié mon fichier jenkinsfile à nouveau)
                    }
                }
            }
        }

        // 🐳 Construction et déploiement Docker
        stage('Docker Build & Up') {
            steps {
                echo " Construction et déploiement des conteneurs Docker..."
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" build"
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" up -d"
            }
        }

        // ✉️ Notification
        stage('Send Notification') {
            steps {
                echo " Envoi de la notification par mail..."
                mail to: "${NOTIFY_EMAIL}",

                     subject: " Jenkins Build Notification",
                     body: "Le build et le déploiement Jenkins se sont terminés avec succès !"

                     subject: "Jenkins Build Notification",
                     body: "✅ Jenkins build and deployment completed successfully."
 (J'ai modifié mon fichier jenkinsfile à nouveau)
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
