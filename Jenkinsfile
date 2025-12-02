pipeline {
    agent any

    tools {
        jdk 'JDK'           // JDK 17
        maven 'Maven'       // Maven 3.x
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📦 Clonage du dépôt GitHub via SSH..."
                git credentialsId: 'idgithub-ssh-key',
                    url: 'git@github.com:bensalahsinda14-hub/game-devops-project.git',
                    branch: 'master'
            }
        }

        stage('Build') {
            steps {
                echo "🔨 Build avec Maven..."
                sh "mvn clean package"
            }
        }

        stage('Tests') {
            steps {
                echo "🧪 Exécution des tests unitaires..."
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Déploiement sur Tomcat..."
                sh '''
                    sudo cp target/*.war /opt/tomcat/latest/webapps/
                    sudo systemctl restart tomcat
                    echo "Application déployée avec succès!"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline terminé avec succès !"
        }
        failure {
            echo "❌ Pipeline échoué !"
        }
    }
}
