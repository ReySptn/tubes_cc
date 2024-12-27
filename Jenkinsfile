pipeline {
    agent any
    environment {
        DOCKER_TAG = ''
    }

    stages {
        stage('SCM') { // Checkout repository first
            steps {
                script {
                    try {
                        git credentialsId: 'tubes_cc', 
                            url: 'https://github.com/ReySptn/tubes_cc.git',
                            branch: 'main'  // Pastikan menggunakan branch main
                    } catch (Exception e) {
                        error "SCM checkout failed: ${e.message}"
                    }
                }
            }
        }

        stage('Set Version') { // Moved after SCM
            steps {
                script {
                    try {
                        def commitHash = bat(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                        env.DOCKER_TAG = commitHash
                    } catch (Exception e) {
                        error "Failed to get commit hash: ${e.message}"
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    try {
                        def gdCheck = bat(script: 'php -m | findstr gd', returnStatus: true)
                        if (gdCheck != 0) {
                            error "PHP GD extension is not enabled. Enable it in php.ini."
                        }
                        bat 'composer install --no-dev --optimize-autoloader'
                    } catch (Exception e) {
                        error "Failed to install dependencies: ${e.message}"
                    }
                }
            }
        }

        stage('Docker Build') {
            when {
                expression { currentBuild.result == null }
            }
            steps {
                script {
                    try {
                        bat "docker build -t reysptan/epos_v1-main:${env.DOCKER_TAG} ."
                    } catch (Exception e) {
                        error "Docker build failed: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        
        success {
            echo 'Pipeline executed successfully!'
            discordSend(
                description: "🎉 Proses build selesai! Docker image berhasil dibuat dengan tag: ${env.DOCKER_TAG}. 🔧 Lihat detail build di Jenkins untuk informasi lebih lanjut.", 
                footer: 'Jenkins CI/CD - Build Berhasil', 
                webhookURL: 'https://discord.com/api/webhooks/1322021893892477010/mA6zP8vWIKIOa23mc2uY6cfMuZtHPuFgmThyvzOWtzCT-iQbNXpMsTJSeZPxEOqry_DU'
            )
        }

        failure {
            echo 'Pipeline failed. Check logs for details.'
            discordSend(
                description: '❌ Build gagal. Silakan cek detail error di Jenkins untuk penyebab kegagalan. ⚠', 
                footer: 'Jenkins CI/CD - Build Gagal', 
                webhookURL: 'https://discord.com/api/webhooks/1322021893892477010/mA6zP8vWIKIOa23mc2uY6cfMuZtHPuFgmThyvzOWtzCT-iQbNXpMsTJSeZPxEOqry_DU'
            )
        }
    }
}
