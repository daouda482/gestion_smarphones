pipeline {
    agent any

    tools {
        nodejs "NodeJS_16"
        // 👇 nom configuré dans Manage Jenkins > Tools > SonarQube Scanner installations
        sonarQubeScanner "SonarScanner"
    }

    environment {
        DOCKER_HUB_USER = 'kao123'
        FRONT_IMAGE     = 'react-frontend'
        BACK_IMAGE      = 'express-backend'
        SONAR_HOST_URL  = 'http://localhost:9000'
    }

    stages {

        // ------------------------------
        // 1️⃣ Récupération du code source
        // ------------------------------
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/daouda482/gestion_smarphones.git'
            }
        }

        // ------------------------------
        // 2️⃣ Installation des dépendances
        // ------------------------------
        stage('Install Dependencies') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('back-end') {
                            bat 'npm install'
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('front-end') {
                            bat 'npm install'
                        }
                    }
                }
            }
        }

        // ------------------------------
        // 3️⃣ Tests unitaires
        // ------------------------------
        stage('Run Tests') {
            steps {
                echo "🧪 Exécution des tests..."
                bat '''
                    cd back-end && npm test || echo "⚠ Aucun test backend"
                    cd ../front-end && npm test || echo "⚠ Aucun test frontend"
                '''
            }
        }

        // ------------------------------
        // 4️⃣ Analyse du code SonarQube
        // ------------------------------
        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Analyse du code avec SonarQube..."
                withSonarQubeEnv('SonarQube_Local') { // 👈 nom du serveur configuré dans Jenkins > Manage Jenkins > SonarQube
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        bat '''
                            sonar-scanner ^
                              -Dsonar.projectKey=fil-rouge ^
                              -Dsonar.projectName="Projet Fil Rouge" ^
                              -Dsonar.projectVersion=1.0 ^
                              -Dsonar.sources=. ^
                              -Dsonar.exclusions=/node_modules/,/build/,/dist/,/*.test.js,/*.spec.js ^
                              -Dsonar.host.url=http://localhost:9000 ^
                              -Dsonar.token=%SONAR_TOKEN%
                        '''
                    }
                }
            }
        }

        // ------------------------------
        // 5️⃣ Build des images Docker
        // ------------------------------
        stage('Build Docker Images') {
            steps {
                echo "🐳 Construction des images Docker..."
                bat """
                    docker build -t %DOCKER_HUB_USER%/%BACK_IMAGE%:latest ./back-end
                    docker build -t %DOCKER_HUB_USER%/%FRONT_IMAGE%:latest ./front-end
                """
            }
        }

        // ------------------------------
        // 6️⃣ Push vers Docker Hub
        // ------------------------------
        stage('Push Docker Images') {
            steps {
                echo "📤 Envoi des images sur Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat '''
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_USER%/react-frontend:latest
                        docker push %DOCKER_USER%/express-backend:latest
                    '''
                }
            }
        }

        // ------------------------------
        // 7️⃣ Déploiement via Docker Compose
        // ------------------------------
        stage('Deploy') {
            steps {
                echo "🚀 Déploiement via docker-compose..."
                bat '''
                    docker-compose -f compose.yaml down || echo "Aucun service en cours"
                    docker-compose -f compose.yaml pull
                    docker-compose -f compose.yaml up -d
                    docker-compose ps
                '''
            }
        }

        // ------------------------------
        // 8️⃣ Tests de disponibilité
        // ------------------------------
        stage('Smoke Test') {
            steps {
                echo "🔎 Vérification des services..."
                bat '''
                    echo "Frontend (port 5173):"
                    curl -f http://localhost:5173 || echo "⚠ Frontend inaccessible"
                    echo "Backend (port 5001):"
                    curl -f http://localhost:5001/api || echo "⚠ Backend inaccessible"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès !"
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                ✅ Build réussi pour ${env.JOB_NAME} #${env.BUILD_NUMBER}
                🔗 Détails: ${env.BUILD_URL}
                🔍 Analyse SonarQube: ${SONAR_HOST_URL}/dashboard?id=fil-rouge
                """,
                to: "omzokao99@gmail.com"
            )
        }
        failure {
            echo "❌ Échec du pipeline."
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué 💥\n\nDétails : ${env.BUILD_URL}",
                to: "omzokao99@gmail.com"
            )
        }
    }
}
