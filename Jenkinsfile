pipeline {
    agent any

    environment {
        APP_NAME = 'game-hub'
        TOMCAT_SERVER = '192.168.17.155'
        TOMCAT_USER = 'sinda'
        WAR_FILE = "target/${APP_NAME}.war"
        DEPLOY_PATH = '/opt/tomcat/webapps'
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo '🔧 Compilation du projet...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo '🚀 Déploiement sur Tomcat...'

                sh """
                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${TOMCAT_USER}@${TOMCAT_SERVER} '
                    sudo systemctl stop tomcat
                    rm -rf ${DEPLOY_PATH}/${APP_NAME}
                    rm -f ${DEPLOY_PATH}/${APP_NAME}.war
                '
                """

                sh """
                scp -o StrictHostKeyChecking=no -i ${SSH_KEY} ${WAR_FILE} ${TOMCAT_USER}@${TOMCAT_SERVER}:${DEPLOY_PATH}/
                """

                sh """
                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${TOMCAT_USER}@${TOMCAT_SERVER} '
                    sudo systemctl start tomcat
                '
                """

                echo '✅ Déploiement terminé avec succès !'
            }
        }
    }
}
