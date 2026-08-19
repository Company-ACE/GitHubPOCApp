pipeline {

    agent any

    environment {

        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        SERVER_NAME = 'TEST'

        SERVER_WORK_DIR = 'C:\\ACE\\servers\\TEST'

        APPLICATION_NAME = 'GitHubPOCApp'

        BAR_NAME = 'GitHubPOCApp.bar'

        HTTP_TEST_URL = 'http://localhost:7800/GitHubPOCFlow'
    }

    stages {

        /*
         * ==========================================================
         * STAGE 1
         * GitHub Checkout
         * ==========================================================
         */

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
                    echo Git Branch:
                    git branch --show-current

                    echo.
                    echo Source Files:
                    dir

                    echo.
                    echo GITHUB CHECKOUT SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 2
         * Validate ACE installation
         * ==========================================================
         */

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
                    ibmint --version

                    echo.
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 3
         * Prepare ACE application
         * ==========================================================
         */

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

                    mkdir "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%"

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

                    copy /Y "%WORKSPACE%\\.project" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

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


        /*
         * ==========================================================
         * STAGE 4
         * Build BAR
         * ==========================================================
         */

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

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR Directory:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%

                    echo.
                    echo BAR File:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

                    echo.
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                        --input-path "%WORKSPACE%\\ace-workspace" ^
                        --output-bar-file "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%"

                    if errorlevel 1 (
                        echo.
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


        /*
         * ==========================================================
         * STAGE 5
         * Verify BAR
         * ==========================================================
         */

        stage('Verify BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 5 - Verify BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo BAR FILE
                    echo ========================================

                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

                    if not exist "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%" (
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo BAR CONTENTS
                    echo ========================================

                    jar tf "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%"

                    if errorlevel 1 (
                        echo ERROR: Unable to read BAR file
                        exit /b 1
                    )

                    echo.
                    echo Checking Application ZIP
                    echo ========================================

                    jar tf "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%" | findstr /I "%APPLICATION_NAME%.appzip"

                    if errorlevel 1 (
                        echo ERROR: %APPLICATION_NAME%.appzip not found
                        exit /b 1
                    )

                    echo.
                    echo BAR CREATED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 6
         * Stop running ACE server
         *
         * This is the important change from your previous pipeline.
         * ==========================================================
         */

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
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\\\ACE\\\\servers\\\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER FOUND'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ACE TEST SERVER IS NOT RUNNING' }"

                    echo.
                    echo Stopping ACE TEST Server
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\\\ACE\\\\servers\\\\TEST*' }; if ($p) { $p | ForEach-Object { Write-Host ('Stopping PID ' + $_.ProcessId); Stop-Process -Id $_.ProcessId -Force } } else { Write-Host 'Server already stopped' }"

                    echo.
                    echo Waiting for server to stop...
                    timeout /t 5 /nobreak

                    echo.
                    echo Verifying server stopped
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\\\ACE\\\\servers\\\\TEST*' }; if ($p) { Write-Host 'ERROR: ACE TEST SERVER STILL RUNNING'; exit 1 } else { Write-Host 'ACE TEST SERVER STOPPED SUCCESSFULLY' }"

                    if errorlevel 1 (
                        echo ERROR: Unable to stop ACE TEST server
                        exit /b 1
                    )

                    echo.
                    echo ACE SERVER STOP SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 7
         * Make sure server work directory exists
         * ==========================================================
         */

        stage('Check ACE Server Work Directory') {

            steps {

                echo '========================================'
                echo 'STAGE 7 - Check ACE Server Work Directory'
                echo '========================================'

                bat '''
                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    if not exist "%SERVER_WORK_DIR%" (

                        echo.
                        echo Server work directory does not exist.
                        echo Creating ACE server work directory...

                        call "%ACE_PROFILE%"

                        mqsicreateworkdir "%SERVER_WORK_DIR%"

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

                    dir "%SERVER_WORK_DIR%"
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 8
         * Deploy BAR
         * ==========================================================
         */

        stage('Deploy BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Deploy BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo DEPLOYING BAR
                    echo ========================================

                    echo Application:
                    echo %APPLICATION_NAME%

                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    call "%ACE_PROFILE%"

                    echo.
                    echo Running ibmint deploy
                    echo ========================================

                    ibmint deploy ^
                        --input-file "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%" ^
                        --work-dir "%SERVER_WORK_DIR%"

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


        /*
         * ==========================================================
         * STAGE 9
         * Start ACE server
         * ==========================================================
         */

        stage('Start ACE Integration Server') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo STARTING ACE TEST SERVER
                    echo ========================================

                    call "%ACE_PROFILE%"

                    echo.
                    echo Starting IntegrationServer...

                    start "" /B "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" ^
                        -w "%SERVER_WORK_DIR%"

                    echo.
                    echo Waiting for ACE server startup...

                    timeout /t 15 /nobreak

                    echo.
                    echo Checking ACE TEST Server
                    echo ========================================

                    powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\\\ACE\\\\servers\\\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER STARTED'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER DID NOT START'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: ACE TEST SERVER FAILED TO START
                        exit /b 1
                    )

                    echo.
                    echo ACE SERVER STARTED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 10
         * Verify server + application deployment
         * ==========================================================
         */

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
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\\\ACE\\\\servers\\\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING' } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

                    if errorlevel 1 (
                        echo ERROR: IntegrationServer is not running
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
                    echo CHECKING APPLICATION
                    echo ========================================

                    dir /S /B "%SERVER_WORK_DIR%" | findstr /I "%APPLICATION_NAME%"

                    echo.
                    echo CHECKING RUNTIME DIRECTORY
                    echo ========================================

                    if exist "%SERVER_WORK_DIR%\\run" (
                        echo ACE runtime directory found
                    ) else (
                        echo WARNING: run directory not found
                    )

                    echo.
                    echo DEPLOYMENT VERIFICATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 11
         * Test actual application endpoint
         *
         * IMPORTANT:
         * Change HTTP_TEST_URL above to your real endpoint.
         * ==========================================================
         */

        stage('Application Test') {

            steps {

                echo '========================================'
                echo 'STAGE 11 - Application Test'
                echo '========================================'

                bat '''
                    echo.
                    echo Testing Application Endpoint
                    echo ========================================

                    echo URL:
                    echo %HTTP_TEST_URL%

                    echo.
                    echo Calling application...

                    curl.exe -i --fail "%HTTP_TEST_URL%"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: Application endpoint test failed
                        exit /b 1
                    )

                    echo.
                    echo APPLICATION ENDPOINT TEST SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        /*
         * ==========================================================
         * STAGE 12
         * Final verification
         * ==========================================================
         */

        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 12 - FINAL VERIFICATION'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo PIPELINE SUMMARY
                    echo ========================================

                    echo.
                    echo Application:
                    echo %APPLICATION_NAME%

                    echo.
                    echo Server:
                    echo %SERVER_NAME%

                    echo.
                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%

                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

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
                    "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\\\ACE\\\\servers\\\\TEST*' }; if ($p) { Write-Host 'ACE TEST SERVER IS RUNNING'; $p | Select-Object ProcessId,CommandLine | Format-List } else { Write-Host 'ERROR: ACE TEST SERVER IS NOT RUNNING'; exit 1 }"

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


    /*
     * ==========================================================
     * POST ACTIONS
     * ==========================================================
     */

    post {

        success {

            echo '========================================'
            echo 'JENKINS BUILD SUCCESS'
            echo '========================================'

            echo "Application: ${env.APPLICATION_NAME}"
            echo "Server: ${env.SERVER_NAME}"
            echo "Build: ${env.BUILD_NUMBER}"
            echo "BAR: ${env.WORKSPACE}\\builds\\${env.BUILD_NUMBER}\\${env.BAR_NAME}"

        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Check the failed stage and ACE logs.'
            echo "ACE Log Directory: ${env.SERVER_WORK_DIR}\\log"

        }

        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
