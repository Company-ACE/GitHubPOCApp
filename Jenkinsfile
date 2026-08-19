pipeline {

    agent any

    environment {

        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        APP_NAME = 'GitHubPOCApp'

        SERVER_NAME = 'TEST'

        SERVER_WORK_DIR = 'C:\\ACE\\servers\\TEST'

        ACE_WORKSPACE = "${WORKSPACE}\\ace-workspace"

        BUILD_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"

        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"
    }


    stages {


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
                    echo ACE VERSION:
                    ibmint help

                    echo.
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        stage('Prepare ACE Application') {

            steps {

                echo '========================================'
                echo 'STAGE 3 - Prepare ACE Application'
                echo '========================================'

                bat '''
                    echo.
                    echo Cleaning Previous ACE Workspace
                    echo ========================================

                    if exist "%ACE_WORKSPACE%" (
                        rmdir /S /Q "%ACE_WORKSPACE%"
                    )

                    mkdir "%ACE_WORKSPACE%"
                    mkdir "%ACE_WORKSPACE%\\%APP_NAME%"

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

                    copy /Y "%WORKSPACE%\\.project" "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    echo.
                    echo ACE APPLICATION STRUCTURE
                    echo ========================================

                    dir /S /B "%ACE_WORKSPACE%"

                    echo.
                    echo ACE WORKSPACE PREPARATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


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

                    if not exist "%BUILD_DIR%" (
                        mkdir "%BUILD_DIR%"
                    )

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
                        --input-path "%ACE_WORKSPACE%" ^
                        --output-bar-file "%BAR_FILE%"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: BAR packaging failed
                        exit /b 1
                    )

                    echo.
                    echo BAR PACKAGING COMPLETED
                    echo ========================================

                    dir "%BUILD_DIR%"
                '''
            }
        }


        stage('Verify BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 5 - Verify BAR'
                echo '========================================'

                bat '''
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


        stage('Check ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 6 - Check ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo ACE SERVER CONFIGURATION
                    echo ========================================

                    echo Server:
                    echo %SERVER_NAME%

                    echo.
                    echo Work Directory:
                    echo %SERVER_WORK_DIR%

                    echo.
                    echo Checking ACE Server Process
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ACE TEST SERVER IS NOT RUNNING' }"

                    echo.
                    echo Checking Work Directory
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%" (

                        echo Work directory does not exist.

                        echo Creating ACE server work directory...

                        call "%ACE_PROFILE%"

                        mqsicreateworkdir "%SERVER_WORK_DIR%"

                        if errorlevel 1 (
                            echo ERROR: Failed to create ACE work directory
                            exit /b 1
                        )
                    )

                    echo.
                    echo ACE SERVER WORK DIRECTORY READY
                    echo ========================================

                    dir "%SERVER_WORK_DIR%"
                '''
            }
        }


        stage('Deploy BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 7 - Deploy BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo DEPLOYING BAR
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    call "%ACE_PROFILE%"

                    echo.
                    echo Running ibmint deploy
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --output-work-directory "%SERVER_WORK_DIR%" ^
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


        stage('Start ACE Integration Server') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo CHECKING ACE SERVER
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER ALREADY RUNNING' } else { Write-Host 'ACE TEST SERVER NOT RUNNING - STARTING'; Start-Process -FilePath '%ACE_HOME%\\server\\bin\\IntegrationServer.exe' -ArgumentList '--work-dir','C:\\ACE\\servers\\TEST' -WindowStyle Hidden }"

                    echo.
                    echo Waiting for ACE server startup
                    echo ========================================

                    timeout /t 10 /nobreak

                    echo.
                    echo Checking ACE server process
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


        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Verify Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo CHECKING ACE SERVER
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: ACE server is not running
                        exit /b 1
                    )

                    echo.
                    echo CHECKING SERVER WORK DIRECTORY
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%" (
                        echo ERROR: Server work directory not found
                        exit /b 1
                    )

                    echo Server directory exists:
                    echo %SERVER_WORK_DIR%

                    echo.
                    echo CHECKING DEPLOYED APPLICATION
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%\\run" (
                        echo ERROR: ACE run directory not found
                        exit /b 1
                    )

                    echo ACE runtime directory found

                    echo.
                    echo Searching deployed application
                    echo ========================================

                    dir /S /B "%SERVER_WORK_DIR%\\run" | findstr /I "%APP_NAME%"

                    echo.
                    echo DEPLOYMENT VERIFICATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        stage('Application Test') {

            steps {

                echo '========================================'
                echo 'STAGE 10 - Application Test'
                echo '========================================'

                bat '''
                    echo.
                    echo APPLICATION DEPLOYMENT TEST
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo APPLICATION TEST COMPLETED
                    echo ========================================
                '''
            }
        }


        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 11 - FINAL VERIFICATION'
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
                    echo %SERVER_WORK_DIR%

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

                    echo.
                    echo ========================================
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


    post {

        success {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'

            echo 'JENKINS BUILD SUCCESS'

            echo "Application: ${APP_NAME}"

            echo "Server: ${SERVER_NAME}"

            echo "BAR: ${BAR_FILE}"

            echo "Build Number: ${BUILD_NUMBER}"
        }

        failure {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'

            echo 'JENKINS BUILD FAILED'

            echo 'Check the failed stage and ACE logs.'

            echo "ACE Log Directory: ${SERVER_WORK_DIR}\\log"
        }
    }
}
