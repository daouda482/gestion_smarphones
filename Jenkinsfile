pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PATH = "C:\\Users\\bmd tech\\Documents\\gestion-smartphones\\docker-compose.yml"
        NOTIFY_EMAIL = "daoudaba679@gmail.com"
        SONARQUBE_ENV = 'SonarQubeServer' // Nom configuré dans Jenkins
        SCANNER_TOOL = 'SonarScanner' // Nom du scanner ajouté dans Global Tool Configuration
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

        // 🔍 Analyse SonarQube
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Injection de l'environnement SonarQube configuré dans Jenkins
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        // Détection automatique du chemin du sonar-scanner
                        def scannerHome = tool name: "${SCANNER_TOOL}", type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                        bat """
                            "${scannerHome}\\bin\\sonar-scanner" ^
                            -Dsonar.projectKey=gestion-smartphone ^
                            -Dsonar.projectName="gestion-martphone" ^
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
                    }
                }
            }
        }

        // 🐳 Construction et déploiement Docker
        stage('Docker Build & Up') {
            steps {
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" build"
                bat "docker-compose -f \"${DOCKER_COMPOSE_PATH}\" up -d"
            }
        }

        // ✉️ Notification
        stage('Send Notification') {
            steps {
                mail to: "${NOTIFY_EMAIL}",
                     subject: "Jenkins Build Notification",
                     body: "✅ Jenkins build and deployment completed successfully."
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
