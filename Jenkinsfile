pipeline {
    agent any

    environment {
        APP_NAME = 'game-hub'
        TOMCAT_USER = 'sinda'
        TOMCAT_HOST = '192.168.17.155'
        TOMCAT_PORT = '8081'
        DEPLOY_PATH = '/opt/tomcat/webapps'
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Récupération du code..."
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                echo "⚙️ Build du projet..."
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Tests') {
            steps {
                echo "🧪 Lancement des tests..."
                sh 'mvn test'
            }
        }

        stage('Package WAR') {
            steps {
                echo "📦 Packaging du WAR..."
                sh 'mv target/*.war ${APP_NAME}.war'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Déploiement sur Tomcat..."

                // Copie du WAR vers le serveur Tomcat
                sh '''
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} "sudo systemctl stop tomcat"
                scp -i ${SSH_KEY} -o StrictHostKeyChecking=no ${APP_NAME}.war ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_PATH}/${APP_NAME}.war
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} "sudo systemctl start tomcat"
                '''
            }
        }
    }

    post {
        success {
            echo "🎉 Déploiement terminé avec succès !"
        }
        failure {
            echo "❌ Échec du pipeline."
        }
    }
}
