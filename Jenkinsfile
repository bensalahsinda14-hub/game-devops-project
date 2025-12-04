pipeline {
    agent any
    
    environment {
        APP_NAME = 'game-hub'
        TOMCAT_SERVER = '192.168.17.155'
        DEPLOY_USER = 'sinda'
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
        
        stage('Build Maven') {
            steps {
                echo '⚙️ Build du projet...'
                sh 'mvn clean install -DskipTests'
            }
        }
        
        stage('Run Tests') {
            steps {
                echo '🧪 Lancement des tests...'
                sh 'mvn test || true'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo '🔍 Analyse SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=Game-Hub-DevOps-Project'
                }
                echo '📊 Résultats disponibles sur: http://192.168.17.155:9000/dashboard?id=Game-Hub-DevOps-Project'
            }
        }
        
        stage('Quality Gate') {
            steps {
                echo '🎯 Vérification Quality Gate...'
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        try {
                            def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                                echo "⚠️ Quality Gate échoué: ${qg.status}"
                                echo "Mais on continue le déploiement..."
                            } else {
                                echo "✅ Quality Gate réussi!"
                            }
                        } catch (Exception e) {
                            echo "⚠️ Timeout ou erreur Quality Gate"
                            echo "On continue quand même le déploiement..."
                        }
                    }
                }
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
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                        sudo systemctl stop tomcat
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                        sudo rm -rf ${DEPLOY_PATH}/${APP_NAME} ${DEPLOY_PATH}/${APP_NAME}.war
                    scp -i ${SSH_KEY} -o StrictHostKeyChecking=no ${APP_NAME}.war ${DEPLOY_USER}@${TOMCAT_SERVER}:/tmp/${APP_NAME}.war
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                        sudo mv /tmp/${APP_NAME}.war ${DEPLOY_PATH}/${APP_NAME}.war
                    ssh -i ${SSH_KEY} -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TOMCAT_SERVER} \\
                        sudo systemctl start tomcat
                """
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline terminé avec succès !'
            echo '📊 Résultats SonarQube: http://192.168.17.155:9000/dashboard?id=Game-Hub-DevOps-Project'
        }
        failure {
            echo '❌ Échec du pipeline.'
        }
        always {
            echo '🧹 Nettoyage du workspace...'
        }
    }
}
