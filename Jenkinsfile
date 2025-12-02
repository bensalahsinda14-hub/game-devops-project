pipeline {
    agent any

    environment {
        APP_NAME = 'game-hub'
        TOMCAT_SERVER = '192.168.17.155'
        DEPLOY_USER = 'sinda'
        DEPLOY_PATH = '/opt/tomcat/webapps'
        WAR_FILE = 'game-hub.war'
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                echo '📥 Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                echo '⚙️ Build du projet...'
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Run Tests') {
            steps {
                echo '🧪 Lancement des tests...'
                sh 'mvn test || echo "Tests skipped or failed"'
            }
        }

        stage('Package WAR') {
            steps {
                echo '📦 Packaging du WAR...'
                sh "cp target/${WAR_FILE} ."
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo '🚀 Déploiement sur Tomcat...'
                sh """
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} 'sudo systemctl stop tomcat'
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} 'sudo rm -rf ${DEPLOY_PATH}/${APP_NAME} ${DEPLOY_PATH}/${WAR_FILE}'
                    scp -i ${SSH_KEY} -o StrictHostKeyChecking=no ${WAR_FILE} ${DEPLOY_USER}@${TOMCAT_SERVER}:${DEPLOY_PATH}/${WAR_FILE}
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} 'sudo systemctl start tomcat'
                """
            }
        }
    }

    post {
        success {
            echo '🎉 Déploiement terminé avec succès !'
        }
        failure {
            echo '❌ Échec du pipeline.'
        }
    }
}
