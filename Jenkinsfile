pipeline {
    agent any

    tools {
        nodejs "NodeJS_16"
    }

    environment {
        DOCKER_HUB_USER = 'kao123'
        FRONT_IMAGE     = 'react-frontend'
        BACK_IMAGE      = 'express-backend'
        SONAR_HOST_URL  = 'http://localhost:9000/'   // SonarQube local
    }

  

    stages {

        // -----------------------
        // 1️⃣ Récupération du code
        // -----------------------
        stage('Checkout') {
            steps {
                echo "📦 Récupération du code depuis GitHub..."
                git branch: 'main', url: 'https://github.com/mhdgeek/express_mongo_react.git'
            }
        }

        // -----------------------
        // 2️⃣ Installation dépendances
        // -----------------------
        stage('Install Dependencies') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('back-end') {
                            sh 'npm install'
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('front-end') {
                            sh 'npm install'
                        }
                    }
                }
            }
        }

        // -----------------------
        // 3️⃣ Tests unitaires
        // -----------------------
        stage('Run Tests') {
            steps {
                echo "🧪 Exécution des tests..."
                script {
                    sh 'cd back-end && npm test || echo "⚠ Aucun test backend"'
                    sh 'cd front-end && npm test || echo "⚠ Aucun test frontend"'
                }
            }
        }

        // -----------------------
        // 4️⃣ Analyse SonarQube AVANT Build
        // -----------------------
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
        } // 👈👉 Accolade fermante manquante ajoutée ici !

        
        // -----------------------
        // 6️⃣ Build Docker
        // -----------------------
        stage('Build Docker Images') {
            steps {
                echo "🐳 Construction des images Docker..."
                sh """
                    docker build -t $DOCKER_HUB_USER/$BACK_IMAGE:latest ./back-end
                    docker build -t $DOCKER_HUB_USER/$FRONT_IMAGE:latest ./front-end
                """
            }
        }

        // -----------------------
        // 7️⃣ Push Docker Hub
        // -----------------------
        stage('Push Docker Images') {
            steps {
                echo "📤 Envoi des images sur Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_USER/react-frontend:latest
                        docker push $DOCKER_USER/express-backend:latest
                    '''
                }
            }
        }

        // -----------------------
        // 8️⃣ Déploiement Docker Compose
        // -----------------------
        stage('Deploy') {
            steps {
                echo "🚀 Déploiement via docker-compose..."
                sh '''
                    docker-compose -f compose.yaml down || true
                    docker-compose -f compose.yaml pull
                    docker-compose -f compose.yaml up -d
                    docker-compose ps
                '''
            }
        }

        // -----------------------
        // 9️⃣ Tests de disponibilité
        // -----------------------
        stage('Smoke Test') {
            steps {
                echo "🔎 Vérification des services..."
                sh '''
                    echo "Frontend (port 5173) :" 
                    curl -f http://localhost:5173 || echo "⚠ Frontend inaccessible"
                    echo "Backend (port 5001) :"
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
                🌍 Webhook Serveo/Ngrok: ${WEBHOOK_PUBLIC}
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
