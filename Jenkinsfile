pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_PATH = "C:\\Users\\bmd tech\\Documents\\gestion-smartphones\\docker-compose.yml"
        NOTIFY_EMAIL = "daoudaba679@gmail.com"
        SONARQUBE_ENV = 'SonarQubeServer' // nom configuré dans Jenkins
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
                    withSonarQubeEnv("${SONARQUBE_ENV}") {
                        // Exemple d’analyse Node.js — à adapter selon ton projet
                        bat 'sonar-scanner -Dsonar.projectKey=gestion-smartphones -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000 -Dsonar.login=your_sonar_token'
                    }
                }
            }
        }

        // 🧩 Quality Gate
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline stopped because quality gate failed: ${qg.status}"
                        }
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
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
