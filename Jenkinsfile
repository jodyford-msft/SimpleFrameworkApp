pipeline {
    agent any 
    stages {
        stage ('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage ('Git Checkout') {
            steps {
                git branch: "master", credentialsId: "github-msm", url: "git@github.com:jodyford-msft/SimpleFrameworkApp.git"
            }
        }
        stage('Restore packages') {
            steps {
                bat "msbuild.exe SimpleFrameworkApp.csproj -t:restore"
                }
         }
        stage('Dotnet Clean') {
            steps {
                bat "msbuild.exe SimpleFrameworkApp.csproj -t:clean"
            }
        }
        stage('Nuget Restore') {
            steps {
                bat "nuget.exe restore SimpleFrameworkApp.csproj -PackagesDirectory Packages"
            }
        }
        stage('Build') {
            steps {
                bat "msbuild.exe SimpleFrameworkApp.csproj /p:Configuration=Release"
            }
        }
        stage("Publish to Azure") {
            steps {
                withCredentials([azureServicePrincipal('sp_jenkins')]) {
                    sh 'az login --service-principal -u a9cbe7ea-8dd2-47e1-9a2b-7f4b14d583fb -p 11e7343b-3d4a-450c-a904-bececf9b84cf -t 72f988bf-86f1-41af-91ab-2d7cd011db47'
                    sh 'az account set -s 9526e7fc-0c5e-4038-9abe-c3dc8b50ac59'
                    # Get FTP publishing profile and query for publish URL and credentials
                    sh 'creds=($(az webapp deployment list-publishing-profiles --name MSM-app --resource-group MSM --query "[?contains(publishMethod, 'FTP')].[publishUrl,userName,userPWD]" --output tsv))'
                    sh 'curl -T index.html -u ${creds[1]}:${creds[2]} ${creds[0]}/'
                }
            }
        }
    }
}
