pipeline {

    agent any

    environment {

        // =========================================================
        // ACE INSTALLATION
        // =========================================================

        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'


        // =========================================================
        // APPLICATION
        // =========================================================

        APP_NAME = 'GitHubPOCApp'


        // =========================================================
        // ACE SERVER
        // =========================================================

        SERVER_NAME = 'TEST'

        SERVER_WORK_DIR = 'C:\\ACE\\servers\\TEST'


        // =========================================================
        // BUILD
        // =========================================================

        BUILD_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"

        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"
    }


    stages {


        // =========================================================
        // STAGE 1
        // =========================================================

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


        // =========================================================
        // STAGE 2
        // =========================================================

        stage('ACE Validation') {

            steps {

                echo '========================================'
                echo 'STAGE 2 - ACE Validation'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo Loading ACE Environment
                    echo ========================================

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


        // =========================================================
        // STAGE 3
        // =========================================================

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


        // =========================================================
        // STAGE 4
        // =========================================================

        stage('Package BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 4 - Package ACE BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo Loading ACE Environment
                    echo ========================================

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


        // =========================================================
        // STAGE 5
        // =========================================================

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


        // =========================================================
        // STAGE 6
        // CREATE ACE SERVER
        // =========================================================

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

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%


                    echo.
                    echo ========================================
                    echo Loading ACE Environment
                    echo ========================================

                    call "%ACE_PROFILE%"


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


                        echo.
                        echo ========================================
                        echo Running mqsicreateworkdir
                        echo ========================================

                        mqsicreateworkdir "%SERVER_WORK_DIR%"

                        if errorlevel 1 (
                            echo ERROR: mqsicreateworkdir failed
                            exit /b 1
                        )


                        echo.
                        echo ========================================
                        echo ACE SERVER WORK DIRECTORY CREATED
                        echo ========================================

                    ) else (

                        echo ACE server work directory already exists.
                    )


                    echo.
                    echo ========================================
                    echo ACE SERVER WORK DIRECTORY READY
                    echo ========================================

                    dir "%SERVER_WORK_DIR%"
                '''
            }
        }


        // =========================================================
        // STAGE 7
        // DEPLOY BAR
        // =========================================================

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

                    call "%ACE_PROFILE%"

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%


                    if not exist "%BAR_FILE%" (
                        echo ERROR: BAR file not found
                        exit /b 1
                    )


                    echo.
                    echo ========================================
                    echo Running ibmint deploy
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --output-work-directory "%SERVER_WORK_DIR%"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: BAR deployment failed
                        exit /b 1
                    )


                    echo.
                    echo ========================================
                    echo BAR DEPLOYMENT SUCCESSFUL
                    echo ========================================

                    echo BAR deployed to:
                    echo %SERVER_WORK_DIR%
                '''
            }
        }


        // =========================================================
        // STAGE 8
        // START ACE SERVER
        // =========================================================

        stage('Start ACE Integration Server') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo STARTING ACE SERVER
                    echo ========================================

                    call "%ACE_PROFILE%"


                    echo.
                    echo ========================================
                    echo Clearing OPENSSL_CONF
                    echo ========================================

                    set "OPENSSL_CONF="


                    echo.
                    echo ========================================
                    echo Checking Existing ACE TEST Server
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER ALREADY RUNNING' } else { exit 1 }"

                    if not errorlevel 1 (
                        echo.
                        echo ACE TEST SERVER IS ALREADY RUNNING
                        goto SERVER_RUNNING
                    )


                    echo.
                    echo ========================================
                    echo Starting IntegrationServer.exe
                    echo ========================================

                    echo Work Directory:
                    echo %SERVER_WORK_DIR%


                    start "ACE-%SERVER_NAME%" /B cmd /c ""%ACE_HOME%\\server\\bin\\IntegrationServer.exe" --work-dir "%SERVER_WORK_DIR%""


                    echo.
                    echo ACE server start command issued.


                    echo.
                    echo ========================================
                    echo Waiting for ACE Server
                    echo ========================================

                    timeout /t 5 /nobreak


                    echo.
                    echo Waiting additional time for initialization...

                    timeout /t 10 /nobreak


                    :SERVER_RUNNING

                    echo.
                    echo ========================================
                    echo ACE SERVER PROCESS CHECK
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: IntegrationServer.exe failed to start
                        echo.
                        echo Check ACE logs under:
                        echo %SERVER_WORK_DIR%\\logs
                        exit /b 1
                    )


                    echo.
                    echo ========================================
                    echo ACE SERVER STARTED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        // =========================================================
        // STAGE 9
        // VERIFY DEPLOYMENT
        // =========================================================

        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Verify Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo CHECKING ACE SERVER
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING' } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: ACE TEST SERVER IS NOT RUNNING
                        exit /b 1
                    )


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
                    echo CHECKING DEPLOYED APPLICATION
                    echo ========================================

                    if exist "%SERVER_WORK_DIR%\\run" (
                        echo ACE runtime directory found
                    ) else (
                        echo WARNING: run directory not found yet
                    )


                    echo.
                    echo ========================================
                    echo ACE SERVER FILE STRUCTURE
                    echo ========================================

                    dir "%SERVER_WORK_DIR%"


                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =========================================================
        // STAGE 10
        // FINAL VERIFICATION
        // =========================================================

        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 10 - FINAL VERIFICATION'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
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

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

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


    // =============================================================
    // POST
    // =============================================================

    post {

        success {

            echo '========================================'
            echo 'JENKINS BUILD SUCCESS'
            echo '========================================'

            echo "Application: ${APP_NAME}"
            echo "Server: ${SERVER_NAME}"
            echo "BAR: ${BAR_FILE}"

            echo '========================================'
        }


        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Check the failed stage in the Jenkins console.'

            echo 'If the failure is related to ACE startup, check:'
            echo 'C:\\ACE\\servers\\TEST\\logs'

            echo '========================================'
        }


        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
