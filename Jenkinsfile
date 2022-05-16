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
                sh 'az login --service-principal -u sp_jenkins -p a9cbe7ea-8dd2-47e1-9a2b-7f4b14d583fb -t 72f988bf-86f1-41af-91ab-2d7cd011db47'
                azureWebAppPublish ([
                    appName: "App-MSM", 
                    azureCredentialsId: "sp_jenkins", 
                    publishType: "file", 
                    resourceGroup: "MSM", 
                    sourceDirectory: "C:/ProgramData/Jenkins/.jenkins/workspace/Sample App/obj/release/"
                ])
            }
        }
    }
}
