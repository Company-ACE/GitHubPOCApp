pipeline {

    agent any

    environment {

        // =====================================================
        // ACE CONFIGURATION
        // =====================================================

        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        APP_NAME = 'GitHubPOCApp'

        SERVER_NAME = 'TEST'

        SERVER_WORKDIR = 'C:\\ACE\\servers\\TEST'

        // Jenkins workspace
        WORKSPACE_DIR = 'C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\TEST'

    }

    stages {

        // =====================================================
        // STAGE 1
        // =====================================================

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

                    echo Git Commit Message:
                    git log -1 --pretty=%%B

                    echo.

                    echo Source Files:
                    dir

                    echo.

                    echo GITHUB CHECKOUT SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 2
        // =====================================================

        stage('ACE Validation') {

            steps {

                echo '========================================'
                echo 'STAGE 2 - ACE Validation'
                echo '========================================'

                bat '''
                    echo.
                    echo Loading ACE Environment
                    echo ========================================

                    call "%ACE_PROFILE%"

                    echo.
                    echo ACE INSTALLATION
                    echo ========================================

                    echo ACE_HOME:
                    echo %ACE_HOME%

                    echo.

                    echo Checking ibmint:
                    where ibmint

                    if errorlevel 1 (
                        echo ERROR: ibmint.exe not found
                        exit /b 1
                    )

                    echo.

                    echo Checking IntegrationServer:
                    where IntegrationServer

                    if errorlevel 1 (
                        echo ERROR: IntegrationServer.exe not found
                        exit /b 1
                    )

                    echo.

                    echo ACE VERSION:
                    ibmint

                    echo.
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 3
        // =====================================================

        stage('Prepare ACE Application') {

            steps {

                echo '========================================'
                echo 'STAGE 3 - Prepare ACE Application'
                echo '========================================'

                bat '''
                    echo.
                    echo Cleaning Previous ACE Workspace
                    echo ========================================

                    if exist "%WORKSPACE%\\ace-workspace" (
                        rmdir /S /Q "%WORKSPACE%\\ace-workspace"
                    )

                    mkdir "%WORKSPACE%\\ace-workspace"
                    mkdir "%WORKSPACE%\\ace-workspace\\%APP_NAME%"

                    echo.
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
                    echo Copying ACE Application Files
                    echo ========================================

                    copy /Y "%WORKSPACE%\\.project" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    echo.
                    echo ACE APPLICATION STRUCTURE
                    echo ========================================

                    dir /S /B "%WORKSPACE%\\ace-workspace"

                    echo.
                    echo ACE WORKSPACE PREPARATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 4
        // =====================================================

        stage('Package BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 4 - Package ACE BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo Loading ACE Environment
                    echo ========================================

                    call "%ACE_PROFILE%"

                    echo.
                    echo Creating BAR Directory
                    echo ========================================

                    if not exist "%WORKSPACE%\\builds\\%BUILD_NUMBER%" (
                        mkdir "%WORKSPACE%\\builds\\%BUILD_NUMBER%"
                    )

                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APP_NAME%.bar

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR File:
                    echo %BAR_FILE%

                    echo.
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                      --input-path "%WORKSPACE%\\ace-workspace" ^
                      --output-bar-file "%BAR_FILE%"

                    if errorlevel 1 (
                        echo ERROR: BAR packaging failed
                        exit /b 1
                    )

                    echo.
                    echo BAR PACKAGING COMPLETED
                    echo ========================================

                    dir "%WORKSPACE%\\builds\\%BUILD_NUMBER%"

                '''
            }
        }


        // =====================================================
        // STAGE 5
        // =====================================================

        stage('Verify BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 5 - Verify BAR'
                echo '========================================'

                bat '''
                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APP_NAME%.bar

                    echo.
                    echo BAR FILE
                    echo ========================================

                    echo %BAR_FILE%

                    if not exist "%BAR_FILE%" (
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo BAR CONTENTS
                    echo ========================================

                    jar tf "%BAR_FILE%"

                    if errorlevel 1 (
                        echo ERROR: Unable to read BAR file
                        exit /b 1
                    )

                    echo.
                    echo Checking Application ZIP
                    echo ========================================

                    jar tf "%BAR_FILE%" | findstr /I "%APP_NAME%.appzip"

                    if errorlevel 1 (
                        echo ERROR: %APP_NAME%.appzip not found
                        exit /b 1
                    )

                    echo.
                    echo BAR CREATED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 6
        // =====================================================

        stage('Stop ACE Integration Server') {

            steps {

                echo '========================================'
                echo 'STAGE 6 - Stop ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo Loading ACE Environment
                    echo ========================================

                    call "%ACE_PROFILE%"

                    echo.
                    echo Checking Existing ACE TEST Server
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING' } else { Write-Host 'ACE TEST SERVER IS NOT RUNNING' }"

                    echo.
                    echo Stopping ACE TEST Server
                    echo ========================================

                    ibmint stop --work-dir "%SERVER_WORKDIR%"

                    echo.
                    echo Waiting for ACE server to stop...
                    timeout /T 10 /NOBREAK

                    echo.
                    echo Verifying server stopped
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ERROR: ACE TEST SERVER IS STILL RUNNING'; exit 1 } else { Write-Host 'ACE TEST SERVER STOPPED SUCCESSFULLY' }"

                    if errorlevel 1 (
                        echo ERROR: Failed to stop ACE server
                        exit /b 1
                    )

                    echo.
                    echo ACE SERVER STOP SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 7
        // =====================================================

        stage('Check ACE Server Work Directory') {

            steps {

                echo '========================================'
                echo 'STAGE 7 - Check ACE Server Work Directory'
                echo '========================================'

                bat '''
                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORKDIR%

                    if not exist "%SERVER_WORKDIR%" (

                        echo.
                        echo Server work directory does not exist.
                        echo Creating ACE server work directory...

                        call "%ACE_PROFILE%"

                        mqsicreateworkdir "%SERVER_WORKDIR%"

                        if errorlevel 1 (
                            echo ERROR: Failed to create ACE work directory
                            exit /b 1
                        )

                    ) else (

                        echo.
                        echo ACE server work directory already exists.

                    )

                    echo.
                    echo ACE SERVER WORK DIRECTORY READY
                    echo ========================================

                    dir "%SERVER_WORKDIR%"
                '''
            }
        }


        // =====================================================
        // STAGE 8
        // =====================================================

        stage('Deploy BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Deploy BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo DEPLOYING BAR
                    echo ========================================

                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APP_NAME%.bar

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORKDIR%

                    call "%ACE_PROFILE%"

                    echo.
                    echo Running ibmint deploy
                    echo ========================================

                    ibmint deploy ^
                      --input-bar-file "%BAR_FILE%" ^
                      --output-work-directory "%SERVER_WORKDIR%" ^
                      --restart-all-applications

                    if errorlevel 1 (
                        echo.
                        echo ERROR: BAR deployment failed
                        exit /b 1
                    )

                    echo.
                    echo BAR DEPLOYMENT SUCCESSFUL
                    echo ========================================

                '''
            }
        }


        // =====================================================
        // STAGE 9
        // =====================================================

        stage('Start ACE Integration Server') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo STARTING ACE SERVER
                    echo ========================================

                    call "%ACE_PROFILE%"

                    echo.
                    echo Starting Integration Server
                    echo ========================================

                    start "" /B "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" -w "%SERVER_WORKDIR%"

                    echo.
                    echo Waiting for ACE server startup...
                    timeout /T 15 /NOBREAK

                    echo.
                    echo Checking ACE Server
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: ACE server failed to start
                        exit /b 1
                    )

                    echo.
                    echo ACE SERVER STARTED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 10
        // =====================================================

        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 10 - Verify Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo CHECKING ACE SERVER
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING' } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        exit /b 1
                    )

                    echo.
                    echo CHECKING SERVER WORK DIRECTORY
                    echo ========================================

                    if not exist "%SERVER_WORKDIR%" (
                        echo ERROR: Server work directory not found
                        exit /b 1
                    )

                    echo Server directory exists:
                    echo %SERVER_WORKDIR%

                    echo.
                    echo CHECKING DEPLOYED APPLICATION
                    echo ========================================

                    if not exist "%SERVER_WORKDIR%\\run" (
                        echo ERROR: ACE run directory not found
                        exit /b 1
                    )

                    echo ACE runtime directory found

                    echo.
                    echo APPLICATION FILES
                    echo ========================================

                    dir /S /B "%SERVER_WORKDIR%\\run" | findstr /I "%APP_NAME%"

                    echo.
                    echo DEPLOYMENT VERIFICATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 11
        // =====================================================

        stage('Application Test') {

            steps {

                echo '========================================'
                echo 'STAGE 11 - Application Test'
                echo '========================================'

                bat '''
                    echo.
                    echo ACE APPLICATION TEST
                    echo ========================================

                    echo.
                    echo Build:
                    echo %BUILD_NUMBER%

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Application deployed successfully.
                    echo.
                    echo If the application exposes HTTP endpoint,
                    echo perform your curl/Postman test here.

                    echo.
                    echo APPLICATION TEST STAGE COMPLETED
                    echo ========================================
                '''
            }
        }


        // =====================================================
        // STAGE 12
        // =====================================================

        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 12 - FINAL VERIFICATION'
                echo '========================================'

                bat '''
                    echo.
                    echo PIPELINE SUMMARY
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Server:
                    echo %SERVER_NAME%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORKDIR%

                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APP_NAME%.bar

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

                    echo.
                    echo FINAL ACE SERVER CHECK
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: Final ACE server verification failed
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE CI/CD PIPELINE SUCCESSFUL
                    echo ========================================
                '''
            }
        }
    }


    // =========================================================
    // POST ACTIONS
    // =========================================================

    post {

        success {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'

            echo 'JENKINS BUILD SUCCESS'

            echo "Application: ${env.APP_NAME}"

            echo "Server: ${env.SERVER_NAME}"

            echo "BAR: ${env.WORKSPACE}\\builds\\${env.BUILD_NUMBER}\\${env.APP_NAME}.bar"

            echo "Build Number: ${env.BUILD_NUMBER}"

            echo '========================================'
        }

        failure {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'

            echo 'JENKINS BUILD FAILED'

            echo 'Check the failed stage and ACE logs.'

            echo "ACE Log Directory: ${env.SERVER_WORKDIR}\\log"

            echo '========================================'
        }
    }
}
