pipeline {
    agent { label 'windows' }
    environment {
        DOTNET_SDK_VERSION = '8.0'
        TEST_IDENTITY_SERVICE = false
        TEST_STOREMANAGEMENT_SERVICE = false 
    }
    
    tools{
        nodejs 'NodeJS_20.13.1'
    }

    stages {
        stage('Restore') {
            steps {
                dir('Dev')  {
                    bat 'dotnet restore Epm.EStore.sln'
                }
            }
        }
        stage('Build') {
            steps {
                dir('Dev') {
                    bat 'dotnet build Epm.EStore.sln --configuration Release'
                }
            }
        }
         stage('Angular Install dependencies') {
            steps {
                dir('Dev/Epm.EStore.AdminUI/epm.estore.adminui.client'){
                    bat 'npm install'
                }
            }
        }
        stage('Angular Build') {
            steps {
             dir('Dev/Epm.EStore.AdminUI/epm.estore.adminui.client') {
                    bat 'npm run build'
                }
            }
        }
         stage('Angular Test'){
            steps{
                dir('Dev/Epm.EStore.AdminUI/epm.estore.adminui.client'){
                    bat 'ng test --watch=false --browsers=ChromeHeadlessCI --code-coverage'
                }
            }
        }

        stage('Test'){
           steps{
                 dir('Dev'){
                    bat 'FOR /R %%G IN (TestResults) DO IF EXIST "%%G" RMDIR /S /Q "%%G"'
                    bat 'dotnet test Epm.EStore.sln --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover' 
                }
            }
        }


        stage('SonarQube analysis') {
           environment {
            def scannerHome = tool 'SonarQube Scanner'
           }
           steps {
             withSonarQubeEnv('SonarHyd') {
               bat "$scannerHome/bin/sonar-scanner.bat"
             }
           }
        }

    }
    post {
        always {
            script {
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    currentBuild.result = 'FAILURE'
                    error "Quality Gate failed: ${qg.status}"
                }
            }
        }
    }
    
}
 
