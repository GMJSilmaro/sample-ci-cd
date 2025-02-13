pipeline {
    agent any

    environment {
        DEPLOY_PATH = 'C:\\Pixelcare\\app\\'
        CREDENTIALS_ID = 'windows-server-creds'
        WIN_IP = '47.250.53.233'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy and Build') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: env.CREDENTIALS_ID,
                        usernameVariable: 'USER',
                        passwordVariable: 'PASSWORD'
                    )]) {
                        // Step 1: Copy files to Windows server
                        pwsh """
                            \$password = ConvertTo-SecureString '$env:PASSWORD' -AsPlainText -Force
                            \$creds = New-Object -TypeName System.Management.Automation.PSCredential('$env:USER', \$password)
                            
                            # Verify destination path
                            if (-not (Test-Path -Path "$env:DEPLOY_PATH" -Credential \$creds)) {
                                New-Item -Path "$env:DEPLOY_PATH" -ItemType Directory -Credential \$creds
                            }
                            
                            # Copy files (exclude node_modules)
                            Get-ChildItem -Path . -Exclude 'node_modules' | 
                                Copy-Item -Destination "$env:DEPLOY_PATH" -Recurse -Force -Credential \$creds -Verbose
                        """
                        
                        // Step 2: Install dependencies and restart app
                        bat """
                            ssh $env:USER@$env:WIN_IP ^"
                                cd $env:DEPLOY_PATH
                                set PM2_HOME=C:\\Users\\$env:USER\\.pm2
                                pnpm install --frozen-lockfile
                                pm2 describe app.js > nul
                                if %errorlevel% equ 0 (
                                    pm2 restart app.js
                                ) else (
                                    pm2 start app.js
                                )
                                pm2 save
                            ^"
                        """
                    }
                }
            }
        }
    }
}