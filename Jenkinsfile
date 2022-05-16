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
        stage('Build and Publish') {
            steps {
                bat "msbuild.exe SimpleFrameworkApp.csproj /p:Configuration=Release"
            }
        }
    }
}
