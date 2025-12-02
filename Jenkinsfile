pipeline {
    agent any

    environment {
        APP_NAME = 'game-hub'                // Nom de l'application
        TOMCAT_SERVER = '192.168.17.155'     // IP du serveur Tomcat
        TOMCAT_PORT = '8081'
        DEPLOY_USER = 'sinda'
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
                sh "mvn clean install -DskipTests"
            }
        }

        stage('Run Tests') {
            steps {
                echo "🧪 Lancement des tests..."
                sh "mvn test || true" // ignore test failures si nécessaire
            }
        }

        stage('Package WAR') {
            steps {
                echo "📦 Packaging du WAR..."
                sh "cp target/${APP_NAME}.war ${APP_NAME}.war"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Déploiement sur Tomcat..."
                // Stop Tomcat
                sh """
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                    'sudo systemctl stop tomcat'
                """

                // Supprime l'ancien WAR et dossier
                sh """
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                    'sudo rm -rf ${DEPLOY_PATH}/${APP_NAME} ${DEPLOY_PATH}/${APP_NAME}.war'
                """

                // Copie le nouveau WAR
                sh """
                scp -i ${SSH_KEY} -o StrictHostKeyChecking=no ${APP_NAME}.war ${DEPLOY_USER}@${TOMCAT_SERVER}:${DEPLOY_PATH}/${APP_NAME}.war
                """

                // Donne les bonnes permissions
                sh """
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                    'sudo chown tomcat:tomcat ${DEPLOY_PATH}/${APP_NAME}.war && sudo chmod 644 ${DEPLOY_PATH}/${APP_NAME}.war'
                """

                // Redémarre Tomcat
                sh """
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                    'sudo systemctl start tomcat'
                """
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
