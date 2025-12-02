pipeline {
    agent any
    
    environment {
        APP_NAME = 'game-hub'
        TOMCAT_SERVER = '192.168.17.155'
        TOMCAT_USER = 'sinda'
        TOMCAT_PATH = '/opt/tomcat/webapps'
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'
    }
    
    stages {
        stage('Checkout SCM') {
            steps {
                echo '📥 Récupération du code depuis GitHub...'
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [], 
                    userRemoteConfigs: [[
                        url: 'git@github.com:bensalahsinda14-hub/game-devops-project.git', 
                        credentialsId: 'github-ssh-key'
                    ]]
                ])
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
                sh 'mvn test'
            }
        }

        stage('Package WAR') {
            steps {
                echo '📦 Packaging du WAR...'
                sh 'cp target/game-hub.war .'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo '🚀 Déploiement sur Tomcat...'
                sh """
                # Stop Tomcat
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} sudo systemctl stop tomcat

                # Copier WAR dans /tmp
                scp -i ${SSH_KEY} -o StrictHostKeyChecking=no game-hub.war ${TOMCAT_USER}@${TOMCAT_SERVER}:/tmp/game-hub.war

                # Déplacer WAR dans webapps avec sudo
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} sudo mv /tmp/game-hub.war ${TOMCAT_PATH}/game-hub.war

                # Redémarrer Tomcat
                ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} sudo systemctl start tomcat
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
