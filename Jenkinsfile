pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_SERVER = 'C:\\ACE\\servers\\TEST'
        APP_NAME = 'GitHubPOCApp'
        WORKSPACE_ACE = "${WORKSPACE}\\ace-workspace"
        BAR_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"
        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"
    }

    stages {

        // ============================================================
        // STAGE 1 - CHECKOUT
        // ============================================================
        stage('Checkout') {
            steps {
                echo '========================================'
                echo 'STAGE 1 - GitHub Checkout'
                echo '========================================'

                bat '''
                echo.
                echo Jenkins Workspace:
                echo %WORKSPACE%

                echo.
                echo Git Commit:
                git rev-parse HEAD

                echo.
                echo Source Files:
                dir

                echo.
                echo ========================================
                echo GITHUB CHECKOUT SUCCESSFUL
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 2 - ACE VALIDATION
        // ============================================================
        stage('ACE Validation') {
            steps {
                echo '========================================'
                echo 'STAGE 2 - ACE Validation'
                echo '========================================'

                bat '''
                call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                echo.
                echo ========================================
                echo ACE INSTALLATION
                echo ========================================

                echo ACE_HOME:
                echo %ACE_HOME%

                echo.
                echo Checking ibmint:
                if not exist "%ACE_HOME%\\server\\bin\\ibmint.exe" (
                    echo ERROR: ibmint.exe not found
                    exit /b 1
                )

                echo %ACE_HOME%\\server\\bin\\ibmint.exe

                echo.
                echo Checking IntegrationServer:
                if not exist "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" (
                    echo ERROR: IntegrationServer.exe not found
                    exit /b 1
                )

                echo %ACE_HOME%\\server\\bin\\IntegrationServer.exe

                echo.
                echo ACE Version:
                "%ACE_HOME%\\server\\bin\\ibmint.exe" version

                echo.
                echo ========================================
                echo ACE VALIDATION SUCCESSFUL
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 3 - PREPARE ACE APPLICATION
        // ============================================================
        stage('Prepare ACE Application') {
            steps {
                echo '========================================'
                echo 'STAGE 3 - Prepare ACE Application'
                echo '========================================'

                bat '''
                echo.
                echo ========================================
                echo Cleaning Previous ACE Workspace
                echo ========================================

                if exist "%WORKSPACE_ACE%" (
                    rmdir /S /Q "%WORKSPACE_ACE%"
                )

                mkdir "%WORKSPACE_ACE%"
                mkdir "%WORKSPACE_ACE%\\%APP_NAME%"

                echo.
                echo ========================================
                echo Checking ACE Application Files
                echo ========================================

                if not exist "%WORKSPACE%\\.project" (
                    echo ERROR: .project not found
                    exit /b 1
                )

                if not exist "%WORKSPACE%\\application.descriptor" (
                    echo ERROR: application.descriptor not found
                    exit /b 1
                )

                if not exist "%WORKSPACE%\\*.msgflow" (
                    echo ERROR: msgflow file not found
                    exit /b 1
                )

                echo.
                echo ========================================
                echo Copying ACE Application Files
                echo ========================================

                copy /Y "%WORKSPACE%\\.project" "%WORKSPACE_ACE%\\%APP_NAME%\\"
                copy /Y "%WORKSPACE%\\application.descriptor" "%WORKSPACE_ACE%\\%APP_NAME%\\"
                copy /Y "%WORKSPACE%\\*.msgflow" "%WORKSPACE_ACE%\\%APP_NAME%\\"
                copy /Y "%WORKSPACE%\\*.esql" "%WORKSPACE_ACE%\\%APP_NAME%\\"

                echo.
                echo ========================================
                echo ACE APPLICATION STRUCTURE
                echo ========================================

                dir /S /B "%WORKSPACE_ACE%"

                echo.
                echo ========================================
                echo ACE WORKSPACE PREPARATION SUCCESSFUL
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 4 - PACKAGE BAR
        // ============================================================
        stage('Package BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 4 - Package ACE BAR'
                echo '========================================'

                bat '''
                call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                echo.
                echo ========================================
                echo Creating BAR Directory
                echo ========================================

                if not exist "%BAR_DIR%" (
                    mkdir "%BAR_DIR%"
                )

                echo.
                echo Build Number:
                echo %BUILD_NUMBER%

                echo.
                echo BAR Directory:
                echo %BAR_DIR%

                echo.
                echo BAR File:
                echo %BAR_FILE%

                echo.
                echo ========================================
                echo Running ibmint package
                echo ========================================

                ibmint package ^
                    "%WORKSPACE_ACE%\\%APP_NAME%" ^
                    --output-file "%BAR_FILE%"

                if errorlevel 1 (
                    echo ERROR: BAR packaging failed
                    exit /b 1
                )

                echo.
                echo ========================================
                echo BAR PACKAGING COMPLETED
                echo ========================================

                dir "%BAR_DIR%"
                '''
            }
        }

        // ============================================================
        // STAGE 5 - VERIFY BAR
        // ============================================================
        stage('Verify BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 5 - Verify BAR'
                echo '========================================'

                bat '''
                echo.
                echo ========================================
                echo BAR FILE
                echo ========================================

                echo %BAR_FILE%

                if not exist "%BAR_FILE%" (
                    echo ERROR: BAR file does not exist
                    exit /b 1
                )

                echo.
                echo ========================================
                echo BAR CONTENTS
                echo ========================================

                jar tf "%BAR_FILE%"

                if errorlevel 1 (
                    echo ERROR: Unable to read BAR file
                    exit /b 1
                )

                echo.
                echo ========================================
                echo Checking Application ZIP
                echo ========================================

                jar tf "%BAR_FILE%" | findstr /I "%APP_NAME%.appzip"

                if errorlevel 1 (
                    echo ERROR: %APP_NAME%.appzip not found
                    exit /b 1
                )

                echo.
                echo ========================================
                echo BAR CREATED SUCCESSFULLY
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 6 - STOP EXISTING ACE SERVER
        // ============================================================
        stage('Stop Existing ACE Server') {
            steps {
                echo '========================================'
                echo 'STAGE 6 - Stop Existing ACE Server'
                echo '========================================'

                bat '''
                echo.
                echo Checking for existing TEST IntegrationServer...

                powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                "$serverPath='C:\\ACE\\servers\\TEST'; ^
                $p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | ^
                Where-Object { $_.CommandLine -like ('*' + $serverPath + '*') }; ^
                if ($p) { ^
                    Write-Host 'Stopping existing TEST ACE Integration Server...'; ^
                    $p | ForEach-Object { ^
                        Write-Host ('Stopping PID ' + $_.ProcessId); ^
                        Stop-Process -Id $_.ProcessId -Force ^
                    } ^
                } else { ^
                    Write-Host 'No existing TEST Integration Server found.' ^
                }"

                echo.
                echo Waiting for ACE server shutdown...

                powershell -NoProfile -Command "Start-Sleep -Seconds 3"

                echo.
                echo Checking TEST IntegrationServer process...

                powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                "$serverPath='C:\\ACE\\servers\\TEST'; ^
                $p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | ^
                Where-Object { $_.CommandLine -like ('*' + $serverPath + '*') }; ^
                if ($p) { ^
                    Write-Host 'ERROR: TEST IntegrationServer is still running'; ^
                    exit 1 ^
                } else { ^
                    Write-Host 'TEST IntegrationServer is stopped.' ^
                }"

                echo.
                echo ========================================
                echo EXISTING SERVER STOPPED
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 7 - DEPLOY BAR
        // ============================================================
        stage('Deploy BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 7 - Deploy BAR'
                echo '========================================'

                bat '''
                echo.
                echo ========================================
                echo Checking ACE Server
                echo ========================================

                if not exist "%ACE_SERVER%" (
                    echo Creating ACE server directory:
                    echo %ACE_SERVER%

                    mkdir "%ACE_SERVER%"
                )

                echo.
                echo ========================================
                echo Deploying BAR
                echo ========================================

                echo BAR:
                echo %BAR_FILE%

                echo Server:
                echo %ACE_SERVER%

                call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                ibmint deploy ^
                    --input-file "%BAR_FILE%" ^
                    --work-dir "%ACE_SERVER%"

                if errorlevel 1 (
                    echo ERROR: BAR deployment failed
                    exit /b 1
                )

                echo.
                echo ========================================
                echo BAR DEPLOYMENT SUCCESSFUL
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 8 - START ACE SERVER
        // ============================================================
        stage('Start ACE Integration Server') {
            steps {
                echo '========================================'
                echo 'STAGE 8 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                echo.
                echo Starting TEST Integration Server...

                if not exist "%ACE_SERVER%" (
                    echo ERROR: ACE Server directory does not exist
                    exit /b 1
                )

                start "ACE-TEST" /B ^
                "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" ^
                --work-dir "%ACE_SERVER%"

                if errorlevel 1 (
                    echo ERROR: Failed to start IntegrationServer
                    exit /b 1
                )

                echo.
                echo Waiting for Integration Server startup...

                powershell -NoProfile -Command "Start-Sleep -Seconds 10"

                echo.
                echo Checking IntegrationServer process...

                powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                "$serverPath='C:\\ACE\\servers\\TEST'; ^
                $p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | ^
                Where-Object { $_.CommandLine -like ('*' + $serverPath + '*') }; ^
                if (-not $p) { ^
                    Write-Host 'ERROR: TEST IntegrationServer is not running'; ^
                    exit 1 ^
                } else { ^
                    Write-Host 'TEST IntegrationServer is running.'; ^
                    $p | Select-Object ProcessId,CommandLine ^
                }"

                echo.
                echo ========================================
                echo ACE INTEGRATION SERVER STARTED
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 9 - VERIFY ACE LISTENER
        // ============================================================
        stage('Verify ACE Listener') {
            steps {
                echo '========================================'
                echo 'STAGE 9 - Verify ACE Listener'
                echo '========================================'

                bat '''
                echo.
                echo Checking ACE listening ports...

                powershell -NoProfile -Command ^
                "Get-NetTCPConnection -State Listen | ^
                Where-Object { $_.OwningProcess -in (Get-Process IntegrationServer -ErrorAction SilentlyContinue).Id } | ^
                Select-Object LocalAddress,LocalPort,OwningProcess"

                echo.
                echo ========================================
                echo ACE LISTENER CHECK COMPLETED
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 10 - VERIFY DEPLOYMENT
        // ============================================================
        stage('Verify Deployment') {
            steps {
                echo '========================================'
                echo 'STAGE 10 - Verify Deployment'
                echo '========================================'

                bat '''
                echo.
                echo ========================================
                echo Checking ACE Server Logs
                echo ========================================

                if exist "%ACE_SERVER%\\logs" (
                    dir "%ACE_SERVER%\\logs"
                )

                echo.
                echo ========================================
                echo Searching for Application
                echo ========================================

                powershell -NoProfile -Command ^
                "$logs = Get-ChildItem -Path '%ACE_SERVER%\\logs' -Recurse -File -ErrorAction SilentlyContinue; ^
                if ($logs) { ^
                    Select-String -Path $logs.FullName -Pattern '%APP_NAME%' -SimpleMatch -ErrorAction SilentlyContinue | ^
                    Select-Object -Last 20 ^
                }"

                echo.
                echo ========================================
                echo DEPLOYMENT VERIFICATION COMPLETED
                echo ========================================
                '''
            }
        }

        // ============================================================
        // STAGE 11 - FINAL VERIFICATION
        // ============================================================
        stage('Final Verification') {
            steps {
                echo '========================================'
                echo 'FINAL VERIFICATION'
                echo '========================================'

                bat '''
                echo.
                echo ========================================
                echo ACE CI/CD PIPELINE SUMMARY
                echo ========================================

                echo Application:
                echo %APP_NAME%

                echo.
                echo Build Number:
                echo %BUILD_NUMBER%

                echo.
                echo Git Commit:
                git rev-parse HEAD

                echo.
                echo BAR File:
                echo %BAR_FILE%

                echo.
                echo ACE Server:
                echo %ACE_SERVER%

                echo.
                echo ========================================
                echo PIPELINE EXECUTION SUCCESSFUL
                echo ========================================
                '''
            }
        }
    }

    // ================================================================
    // POST ACTIONS
    // ================================================================
    post {

        success {
            echo '========================================'
            echo 'JENKINS BUILD SUCCESSFUL'
            echo '========================================'
            echo "Application: ${APP_NAME}"
            echo "Build Number: ${BUILD_NUMBER}"
            echo "BAR: ${BAR_FILE}"
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'
            echo 'Check the failed stage in the Jenkins console.'
            echo '========================================'
        }

        always {
            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
