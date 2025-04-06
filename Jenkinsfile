pipeline {
    agent any
    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'webapijenkinsnaitik457'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/naitikjain25/React-jenkins.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                }
            }
        }

        stage('Terraform Plan & Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform plan -out=tfplan'
                    bat 'terraform apply -auto-approve tfplan'
                }
            }
        }

        stage('Build') {
            steps {
                dir('my-react-app') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

                stage('Deploy') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat "az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%"
                    
                    dir('my-react-app/build') {
                        bat 'powershell Compress-Archive -Path * -DestinationPath ../../publish.zip -Force'
                    }

                    bat 'powershell Expand-Archive -Path publish.zip -DestinationPath zip_contents -Force'
                    bat 'dir zip_contents' // Optional: view contents for debugging

                    bat "az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path publish.zip --type zip --verbose"

                    bat "az logout"
                }
            }
        }

    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
