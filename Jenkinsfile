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
                bat "dotnet restore SimpleFrameworkApp.sln"
                }
         }
        stage('Clean') {
            steps {
                bat "dotnet clean SimpleFrameworkApp.sln"
            }
        }
        stage('Increase version') {
            steps {
                echo "${env.BUILD_NUMBER}"
                    powershell '''
                       $xmlFileName = "<path-to-solution>\\<package-project-name>\\Package.appxmanifest"     
                       [xml]$xmlDoc = Get-Content $xmlFileName
                       $version = $xmlDoc.Package.Identity.Version
                       $trimmedVersion = $version -replace '.[0-9]+$', '.'
                       $xmlDoc.Package.Identity.Version = $trimmedVersion + ${env:BUILD_NUMBER}
                       echo 'New version:' $xmlDoc.Package.Identity.Version
                       $xmlDoc.Save($xmlFileName)
                    '''
                }
            }
        stage('Build') {
            steps {
                bat "dotnet build ${workspace}\\SimpleFrameworkApp.sln"
            }
        }
        stage('Running unit tests') {
            steps {
                bat "dotnet add ${workspace}/<path-to-Unit-testing-project>/<name-of-unit-test-project>.csproj package JUnitTestLogger --version 1.1.0"
                bat "dotnet test ${workspace}/<path-to-Unit-testing-project>/<name-of-unit-test-project>.csproj --logger \"junit;LogFilePath=\"${WORKSPACE}\"/TestResults/1.0.0.\"${env.BUILD_NUMBER}\"/results.xml\" --configuration release --collect \"Code coverage\""
                powershell '''
                  $destinationFolder = \"$env:WORKSPACE/TestResults\"
                  if (!(Test-Path -path $destinationFolder)) {New-Item $destinationFolder -Type Directory}
                  $file = Get-ChildItem -Path \"$env:WORKSPACE/<path-to-Unit-testing-project>/<name-of-unit-test-project>/TestResults/*/*.coverage\"
                  $file | Rename-Item -NewName testcoverage.coverage
                  $renamedFile = Get-ChildItem -Path \"$env:WORKSPACE/<path-to-Unit-testing-project>/<name-of-unit-test-project>/TestResults/*/*.coverage\"
                  Copy-Item $renamedFile -Destination $destinationFolder
                '''            
            }        
        }
        stage('Convert coverage file to xml coverage file') {
            steps {
                bat "<path-to-CodeCoverage.exe>\\CodeCoverage.exe analyze  /output:${WORKSPACE}\\TestResults\\xmlresults.coveragexml  ${WORKSPACE}\\TestResults\\testcoverage.coverage"
            }
        }
        stage('Generate report') {
            steps {
                bat "<path-to-ReportGenerator.exe>\\ReportGenerator.exe -reports:${WORKSPACE}\\TestResults\\xmlresults.coveragexml -targetdir:${WORKSPACE}\\CodeCoverage_${env.BUILD_NUMBER}"
            }
        }
        stage('Publish HTML report') {
            steps {
                publishHTML(target: [allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'CodeCoverage_${BUILD_NUMBER}', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: 'Code Coverage Report'])
            }
        }
    }
}
