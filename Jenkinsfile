pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'reactapijenkinsyashh015'
        TERRAFORM_PATH = 'C:\\Users\\soniy\\Downloads\\terraform_1.11.3_windows_386\\terraform.exe'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/mishra-rd015/react-azure-pipeline.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    bat '"%TERRAFORM_PATH%" init'
                }
            }
        }

        stage('Terraform Plan & Apply') {
            steps {
                dir('terraform') {
                    bat '"%TERRAFORM_PATH%" plan -out=tfplan'
                    bat '"%TERRAFORM_PATH%" apply -auto-approve tfplan'
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('my-react-app') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: env.AZURE_CREDENTIALS_ID,
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID'
                )]) {
                    script {
                        bat '''
                        IF EXIST publish (rmdir /s /q publish)
                        mkdir publish

                        xcopy /s /e /y my-react-app\\build\\* publish\\

                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        powershell Compress-Archive -Path publish\\* -DestinationPath publish.zip -Force
                        az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path publish.zip --type zip --verbose
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
