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
                git credentialsId: 'github-ssh-key',
                    url: 'git@github.com:bensalahsinda14-hub/game-devops-project.git',
                    branch: 'main'
                echo '✅ Code récupéré'
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Construction du projet...'
                sh '''
                    mkdir -p dist/${APP_NAME}

                    # Copier fichiers HTML si existent
                    if ls *.html 1> /dev/null 2>&1; then
                        cp *.html dist/${APP_NAME}/
                    fi

                    # Copier dossiers css, js, images, assets
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
                            sudo rm -rf /opt/tomcat/webapps/game-hub
                            cd /tmp
                            tar -xzf game-hub.tar.gz
                            sudo mv game-hub /opt/tomcat/webapps/
                            sudo chown -R tomcat:tomcat /opt/tomcat/webapps/game-hub
                            rm -f /tmp/game-hub.tar.gz
ENDSSH
                    '''
                }
                echo "✅ Déploiement terminé"
            }
        }

        stage('Health Check') {
            steps {
                echo '🔍 Vérification de l’application...'
                script {
                    sleep 10
                    def response = sh(
                        script: "curl -s -o /dev/null -w '%{http_code}' http://${TOMCAT_SERVER}:${TOMCAT_PORT}/${APP_NAME}/",
                        returnStdout: true
                    ).trim()

                    if (response == '200') {
                        echo "✅ Application UP et fonctionnelle !"
                        echo "🌐 URL : http://${TOMCAT_SERVER}:${TOMCAT_PORT}/${APP_NAME}/"
                    } else {
                        error "❌ Health check failed (status = ${response})"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline exécuté avec succès !'
        }
        failure {
            echo '❌ Pipeline échoué !'
        }
    }
}
