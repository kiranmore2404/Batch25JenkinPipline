pipeline {
    agent any

    environment {
        SOLUTION_NAME = "Batch25JenkinPipline.sln"
        SONAR_PROJECT_KEY = "Batch25JenkinPipline"
        SONAR_SCANNER_NAME = "SonarScanner for MSBuild"   // must match Jenkins tool name
    }

    stages {

        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Sonar + Build + Test') {
            steps {
                script {

                    // resolve scanner path correctly
                    def scannerHome = tool SONAR_SCANNER_NAME

                    withSonarQubeEnv('MySonarQube') {

                        // ---------- SONAR BEGIN ----------
                        bat """
                        "${scannerHome}\\SonarScanner.MSBuild.exe" begin ^
                          /k:"${SONAR_PROJECT_KEY}" ^
                          /d:sonar.cs.opencover.reportsPaths=TestResults/**/coverage.opencover.xml
                        """

                        // ---------- BUILD ----------
                        bat "dotnet build \"${SOLUTION_NAME}\" --configuration Debug"

                        // ---------- TEST WITH COVERAGE ----------
                        bat """
                        dotnet test \"${SOLUTION_NAME}\" ^
                          --no-build ^
                          --collect:\"XPlat Code Coverage\" ^
                          --results-directory TestResults ^
                          -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover
                        """

                        // ---------- SONAR END ----------
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
    }
}
