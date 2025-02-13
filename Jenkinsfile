pipeline {
    agent any

    environment {
        DEPLOY_PATH = 'C:\\Pixelcare\\app\\'
        CREDENTIALS_ID = 'windows-server-creds' // Jenkins credential ID
        WIN_IP = '47.250.53.233' // Replace with your server IP
    }

    stages {
        // Stage 1: Pull code from GitHub
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Stage 2: Deploy & Build on Windows Server
        stage('Deploy and Build') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.CREDENTIALS_ID,
                        usernameVariable: 'USER',
                        passwordVariable: 'PASSWORD'
                    )]) {
                        // Copy code to Windows server (exclude node_modules)
                        pwsh """
                            \$password = ConvertTo-SecureString -AsPlainText '$env:PASSWORD' -Force
                            \$creds = New-Object -TypeName System.Management.Automation.PSCredential('$env:USER', \$password)
                            
                            # Copy only necessary files (exclude node_modules)
                            Get-ChildItem -Path . -Exclude 'node_modules' | 
                                Copy-Item -Destination "$env:DEPLOY_PATH" -Recurse -Force -Credential \$creds
                        """
                        
                        // Install dependencies and restart app via SSH
                        bat """
                            ssh $env:USER@$env:WIN_IP "
                                cd $env:DEPLOY_PATH
                                pnpm install
                                pm2 restart app.js
                            "
                        """
                    }
                }
            }
        }
    }
}