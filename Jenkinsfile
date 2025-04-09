pipeline {
    agent any

    options {
        timestamps() // Add timestamps to console output
    }

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'reactapijenkinsyashh015'
        TERRAFORM_PATH = 'C:\\Users\\soniy\\Downloads\\terraform_1.11.3_windows_386\\terraform.exe'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/yashhsoni/TerraPipe.git'
            }
        }

        stage('Terraform Init & Validate') {
            steps {
                dir('terraform') {
                    bat '"%TERRAFORM_PATH%" init'
                    bat '"%TERRAFORM_PATH%" validate'
                }
            }
        }

        stage('Terraform Plan & Apply') {
            steps {
                dir('terraform') {
                    catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                        bat '"%TERRAFORM_PATH%" plan -out=tfplan'
                        bat '"%TERRAFORM_PATH%" apply -auto-approve tfplan'
                    }
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

                        powershell Compress-Archive -Path publish\\* -DestinationPath publish.zip

                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%

                        az webapp deployment source config-zip --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src publish.zip
                        '''
                    }
                }
            }
        }
    }
}
