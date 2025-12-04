pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PATH = "C:\\Users\\bmd tech\\Documents\\gestion-smartphones\\docker-compose.yml"
        NOTIFY_EMAIL = "daoudaba679@gmail.com"
        SONARQUBE_ENV = 'SonarQube_Local'
        SONAR_SCANNER = 'SonarQube_Scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Clonage du dépôt Git..."
                git branch: 'main', url: 'https://github.com/daouda482/gestion_smarphones.git'
            }
        }

        stage('Install Backend') {
            steps {
                echo "📦 Installation du backend..."
                dir('gestion-smartphone-backend') {
                    bat 'npm install'
                }
            }
        }

        stage('Install & Build Frontend') {
            steps {
                echo "⚙️ Installation et build du frontend..."
                dir('gestion-smartphone-frontend') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        // stage('Run Frontend Tests & Coverage') {
        //     steps {
        //         echo "🧪 Exécution des tests frontend avec coverage..."
        //         dir('gestion-smartphone-frontend') {
        //             bat 'npm run test:coverage'
        //         }
        //     }
        // }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyse du code avec SonarQube..."
                dir('gestion-smartphone-frontend') {
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        // Le scanner prend automatiquement le coverage généré par Vitest
                        bat """
                            "${tool SONAR_SCANNER}\\bin\\sonar-scanner" ^
                            -Dsonar.projectKey=gestion-smartphones ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONAR_HOST_URL% ^
                            -Dsonar.login=%SONAR_TOKEN% ^
                            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                        """
                    }
                }
            }
        }

      stage('Quality Gate') {
    steps {
        script {
            // On met 10 minutes pour être sûr que SonarQube ait le temps de calculer
            timeout(time: 10, unit: 'MINUTES') { 
                def qg = waitForQualityGate()
                echo "Quality Gate Status : ${qg.status}"
                if (qg.status != 'OK') {
                    error "❌ Build stoppé — Quality Gate échoué (${qg.status})"
                } else {
                    echo "✅ Quality Gate validé !"
                }
            }
        }
    }
}


        stage('Docker Build & Up') {
            steps {
                echo "🐳 Construction et lancement des conteneurs Docker..."
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" build"
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" up -d"
            }
        }

        stage('Send Notification') {
            steps {
                echo "📧 Envoi du mail de notification..."
                mail to: "${NOTIFY_EMAIL}",
                     subject: "Jenkins Build Notification",
                     body: "Le build et le déploiement Jenkins se sont terminés avec succès !"
            }
        }
    }

    post {
        success {
            echo '🎉 Build et déploiement réussis !'
        }
        failure {
            echo '❌ Le build a échoué.'
        }
    }
}
