pipeline {
    agent any
    
    environment {
        APP_NAME = 'game-hub'
        TOMCAT_SERVER = '192.168.17.155'
        TOMCAT_PORT = '8081'
        DEPLOY_USER = 'sinda'
        DEPLOY_PATH = '/opt/tomcat/webapps'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📦 Clonage du dépôt GitHub via SSH...'
                checkout scm
                echo '✅ Code récupéré'
            }
        }
        
        stage('Build') {
            steps {
                echo '🔨 Construction du projet...'
                sh '''
                    mkdir -p dist/${APP_NAME}
                    
                    if ls *.html 1> /dev/null 2>&1; then
                        cp *.html dist/${APP_NAME}/
                    fi
                    
                    for dir in css js images assets; do
                        if [ -d "$dir" ]; then
                            cp -r "$dir" dist/${APP_NAME}/
                        fi
                    done
                    
                    echo "✅ Build terminé !"
                    ls -la dist/${APP_NAME}/
                '''
            }
        }
        
        stage('Package') {
            steps {
                echo '📦 Création du package TAR...'
                sh '''
                    cd dist
                    tar -czf ${APP_NAME}.tar.gz ${APP_NAME}/
                    echo "✅ Package créé : dist/${APP_NAME}.tar.gz"
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo "🚀 Déploiement sur le serveur Tomcat (${TOMCAT_SERVER})..."
                sshagent(['ssh-tomcat']) {
                    sh '''
                        scp dist/${APP_NAME}.tar.gz ${DEPLOY_USER}@${TOMCAT_SERVER}:/tmp/
                        
                        ssh ${DEPLOY_USER}@${TOMCAT_SERVER} << 'ENDSSH'
                            sudo rm -rf ${DEPLOY_PATH}/${APP_NAME}
                            cd /tmp
                            tar -xzf ${APP_NAME}.tar.gz
                            sudo mv ${APP_NAME} ${DEPLOY_PATH}/
                            sudo chown -R tomcat:tomcat ${DEPLOY_PATH}/${APP_NAME}
                            rm -f /tmp/${APP_NAME}.tar.gz
ENDSSH
                        
                        echo "✅ Déploiement terminé"
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo '🩺 Vérification de l\'application...'
                script {
                    sleep 10
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' http://${TOMCAT_SERVER}:${TOMCAT_PORT}/${APP_NAME}/",
                        returnStdout: true
                    ).trim()
                    
                    if (response == '200') {
                        echo "✅ Application OK !"
                        echo "🌐 http://${TOMCAT_SERVER}:${TOMCAT_PORT}/${APP_NAME}/"
                    } else {
                        error "❌ Health check failed, HTTP code: ${response}"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline réussi !'
        }
        failure {
            echo '❌ Pipeline échoué !'
        }
        always {
            echo '📊 Pipeline terminé - Vérifiez les logs pour plus de détails'
        }
    }
}
