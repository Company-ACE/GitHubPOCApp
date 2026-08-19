pipeline {

    agent any

    environment {

        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        APP_NAME = 'GitHubPOCApp'

        SERVER_NAME = 'TEST'

        SERVER_WORK_DIR = 'C:\\ACE\\servers\\TEST'

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
                    echo Source Files:
                    dir

                    echo.
                    echo ========================================
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
                    call "%ACE_PROFILE%"

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

                    echo ibmint.exe found

                    echo.
                    echo Checking IntegrationServer:

                    if not exist "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" (
                        echo ERROR: IntegrationServer.exe not found
                        exit /b 1
                    )

                    echo IntegrationServer.exe found

                    echo.
                    echo ========================================
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
                    echo ========================================
                    echo Cleaning Previous ACE Workspace
                    echo ========================================

                    if exist "%WORKSPACE%\\ace-workspace" (
                        rmdir /S /Q "%WORKSPACE%\\ace-workspace"
                    )

                    mkdir "%WORKSPACE%\\ace-workspace"
                    mkdir "%WORKSPACE%\\ace-workspace\\%APP_NAME%"

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

                    copy /Y "%WORKSPACE%\\.project" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    echo.
                    echo ========================================
                    echo ACE APPLICATION STRUCTURE
                    echo ========================================

                    dir /S /B "%WORKSPACE%\\ace-workspace"

                    echo.
                    echo ========================================
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
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo Creating BAR Directory
                    echo ========================================

                    if not exist "%BUILD_DIR%" (
                        mkdir "%BUILD_DIR%"
                    )

                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR Directory:
                    echo %BUILD_DIR%

                    echo.
                    echo BAR File:
                    echo %BAR_FILE%

                    echo.
                    echo ========================================
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
                    echo ========================================
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


        stage('Check/Create ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 6 - Check/Create ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo ACE SERVER CONFIGURATION
                    echo ========================================

                    echo Server Name:
                    echo %SERVER_NAME%

                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    echo.
                    echo ========================================
                    echo Loading ACE Environment
                    echo ========================================

                    call "%ACE_PROFILE%"

                    if errorlevel 1 (
                        echo ERROR: ACE profile failed
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo CLEARING OPENSSL_CONF
                    echo ========================================

                    set "OPENSSL_CONF="

                    echo OPENSSL_CONF=%OPENSSL_CONF%

                    echo.
                    echo ========================================
                    echo Creating Base Directories
                    echo ========================================

                    if not exist "C:\\ACE" (
                        mkdir "C:\\ACE"
                    )

                    if not exist "C:\\ACE\\servers" (
                        mkdir "C:\\ACE\\servers"
                    )

                    echo.
                    echo ========================================
                    echo Checking ACE Server Work Directory
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%" (

                        echo Server work directory does not exist.

                        echo.
                        echo Creating:
                        echo %SERVER_WORK_DIR%

                        mkdir "%SERVER_WORK_DIR%"

                        if errorlevel 1 (
                            echo ERROR: Unable to create server directory
                            exit /b 1
                        )

                        echo.
                        echo ========================================
                        echo Running mqsicreateworkdir
                        echo ========================================

                        set "OPENSSL_CONF="

                        mqsicreateworkdir "%SERVER_WORK_DIR%"

                        if errorlevel 1 (
                            echo.
                            echo ERROR: mqsicreateworkdir failed
                            echo.
                            echo Checking OPENSSL_CONF:
                            echo [%OPENSSL_CONF%]
                            exit /b 1
                        )

                        echo.
                        echo ========================================
                        echo ACE SERVER WORK DIRECTORY CREATED
                        echo ========================================

                    ) else (

                        echo.
                        echo ========================================
                        echo ACE SERVER WORK DIRECTORY EXISTS
                        echo ========================================

                        echo %SERVER_WORK_DIR%
                    )

                    echo.
                    echo ========================================
                    echo CHECKING ACE SERVER PROCESS
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*%SERVER_WORK_DIR%*' }; if ($p) { Write-Host 'TEST ACE Integration Server is already running.' } else { Write-Host 'TEST ACE Integration Server is not running.' }"

                    echo.
                    echo ========================================
                    echo STARTING ACE SERVER IF REQUIRED
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*%SERVER_WORK_DIR%*' }; if (-not $p) { Write-Host 'Starting TEST ACE Integration Server...'; Start-Process -FilePath '%ACE_HOME%\\server\\bin\\IntegrationServer.exe' -ArgumentList '--work-dir','%SERVER_WORK_DIR%' -WindowStyle Hidden } else { Write-Host 'TEST ACE Integration Server already running.' }"

                    echo.
                    echo Waiting for ACE server initialization...

                    timeout /t 15 /nobreak >nul

                    echo.
                    echo ========================================
                    echo VERIFYING ACE SERVER PROCESS
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*%SERVER_WORK_DIR%*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING' } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: ACE Integration Server failed to start
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER IS RUNNING
                    echo ========================================
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
                    echo ========================================
                    echo DEPLOYING BAR
                    echo ========================================

                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    call "%ACE_PROFILE%"

                    if errorlevel 1 (
                        echo ERROR: ACE profile failed
                        exit /b 1
                    )

                    set "OPENSSL_CONF="

                    echo.
                    echo Running ibmint deploy...

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
                    echo ========================================
                    echo BAR DEPLOYMENT SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Verify Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo CHECKING SERVER WORK DIRECTORY
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%" (
                        echo ERROR: Server work directory not found
                        exit /b 1
                    )

                    echo Server directory exists:
                    echo %SERVER_WORK_DIR%

                    echo.
                    echo ========================================
                    echo CHECKING ACE SERVER PROCESS
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*%SERVER_WORK_DIR%*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING' } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: IntegrationServer.exe is not running
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'FINAL VERIFICATION'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo PIPELINE SUMMARY
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo Server:
                    echo %SERVER_NAME%

                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    echo BAR:
                    echo %BAR_FILE%

                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

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
            echo 'JENKINS BUILD SUCCESS'
            echo '========================================'

            echo "Application: ${APP_NAME}"
            echo "Server: ${SERVER_NAME}"
            echo "BAR: ${BAR_FILE}"
        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Check the failed stage in the Jenkins console.'
        }

        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
