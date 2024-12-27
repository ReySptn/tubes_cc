pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'epos_v1-main:${env.DOCKER_TAG}' // Ganti dengan nama dan tag image Docker Anda
        DISCORD_WEBHOOK = 'https://discord.com/api/webhooks/1322021893892477010/mA6zP8vWIKIOa23mc2uY6cfMuZtHPuFgmThyvzOWtzCT-iQbNXpMsTJSeZPxEOqry_DU'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Melakukan checkout kode...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Membangun image Docker...'
                sh '''
                    docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Mengirim image Docker ke repository...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Notify Discord') {
            steps {
                echo 'Mengirim notifikasi ke Discord...'
                script {
                    def payload = '{"content": "Pipeline berhasil dijalankan! 🎉 Docker image telah dibuat dan di-push: ' + env.DOCKER_IMAGE + '"}'
                    sh """
                        curl -H "Content-Type: application/json" -X POST -d '${payload}' ${DISCORD_WEBHOOK}
                    """
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline gagal. Mengirim notifikasi ke Discord...'
            script {
                def payload = '{"content": "⚠️ Pipeline gagal! Mohon periksa log error di Jenkins."}'
                sh """
                    curl -H "Content-Type: application/json" -X POST -d '${payload}' ${DISCORD_WEBHOOK}
                """
            }
        }
    }
}
