pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        ACE_WORK_DIR = 'C:\\ACE\\servers\\TEST'

        APP_NAME = 'GitHubPOCApp'

        BAR_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"
        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"

        ACE_SERVER_EXE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\IntegrationServer.exe'
    }

    stages {

        // =========================================================
        // 1. CHECKOUT
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
                    echo Git Commit Message:
                    git log -1 --pretty=%%B

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


        // =========================================================
        // 2. ACE VALIDATION
        // =========================================================

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
                    echo ACE Installation:
                    echo %ACE_HOME%

                    echo.
                    echo Checking ibmint:
                    where ibmint

                    if errorlevel 1 (
                        echo ERROR: ibmint not found
                        exit /b 1
                    )

                    echo.
                    echo Checking IntegrationServer:
                    if not exist "%ACE_SERVER_EXE%" (
                        echo ERROR: IntegrationServer.exe not found
                        exit /b 1
                    )

                    echo.
                    echo ACE VERSION:
                    ibmint --help

                    echo.
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // =========================================================
        // 3. PREPARE APPLICATION
        // =========================================================

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

                    copy /Y "%WORKSPACE%\\.project" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" ^
                        "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\"

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


        // =========================================================
        // 4. PACKAGE BAR
        // =========================================================

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

                    if exist "%BAR_DIR%" (
                        rmdir /S /Q "%BAR_DIR%"
                    )

                    mkdir "%BAR_DIR%"

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

                    dir "%BAR_DIR%"
                '''
            }
        }


        // =========================================================
        // 5. VERIFY BAR
        // =========================================================

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


        // =========================================================
        // 6. CHECK ACE SERVER
        // =========================================================

        stage('Check ACE Server') {
            steps {

                echo '========================================'
                echo 'STAGE 6 - Check ACE Integration Server'
                echo '========================================'

                bat '''
                    echo.
                    echo Checking ACE Integration Server
                    echo ========================================

                    tasklist /FI "IMAGENAME eq IntegrationServer.exe" | find /I "IntegrationServer.exe" >nul

                    if errorlevel 1 (

                        echo.
                        echo ACE TEST SERVER IS NOT RUNNING
                        echo.

                        echo Starting ACE Integration Server...
                        echo ========================================

                        start "ACE-TEST" /B "%ACE_SERVER_EXE%" ^
                            --name TEST ^
                            --work-dir "%ACE_WORK_DIR%" ^
                            > "%ACE_WORK_DIR%\\log\\jenkins-startup.log" 2>&1

                        echo.
                        echo Waiting for ACE server startup...

                        timeout /T 15 /NOBREAK

                        tasklist /FI "IMAGENAME eq IntegrationServer.exe" | find /I "IntegrationServer.exe" >nul

                        if errorlevel 1 (
                            echo ERROR: ACE Integration Server failed to start
                            echo Check:
                            echo %ACE_WORK_DIR%\\log
                            exit /b 1
                        )

                        echo.
                        echo ACE TEST SERVER STARTED SUCCESSFULLY

                    ) else (

                        echo.
                        echo ACE TEST SERVER IS ALREADY RUNNING

                    )

                    echo.
                    echo ACE SERVER READY
                    echo ========================================
                '''
            }
        }


        // =========================================================
        // 7. DEPLOY BAR
        // =========================================================

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
                    echo %ACE_WORK_DIR%

                    echo.
                    echo Loading ACE Environment
                    echo ========================================

                    call "%ACE_PROFILE%"

                    echo.
                    echo Running ibmint deploy
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --output-work-directory "%ACE_WORK_DIR%" ^
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


        // =========================================================
        // 8. VERIFY SERVER
        // =========================================================

        stage('Verify Deployment') {
            steps {

                echo '========================================'
                echo 'STAGE 8 - Verify Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo Checking ACE Integration Server
                    echo ========================================

                    tasklist /FI "IMAGENAME eq IntegrationServer.exe" | find /I "IntegrationServer.exe" >nul

                    if errorlevel 1 (
                        echo ERROR: ACE Integration Server is not running
                        exit /b 1
                    )

                    echo.
                    echo ACE Integration Server is RUNNING

                    echo.
                    echo Checking Application Deployment
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Checking server logs:
                    echo %ACE_WORK_DIR%\\log

                    if not exist "%ACE_WORK_DIR%\\log" (
                        echo ERROR: ACE log directory not found
                        exit /b 1
                    )

                    echo.
                    echo DEPLOYMENT VERIFICATION COMPLETED
                    echo ========================================
                '''
            }
        }


        // =========================================================
        // 9. APPLICATION TEST
        // =========================================================

        stage('Application Test') {
            steps {

                echo '========================================'
                echo 'STAGE 9 - Application Test'
                echo '========================================'

                bat '''
                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo ACE Server:
                    echo %ACE_WORK_DIR%

                    echo.
                    echo Application test placeholder.
                    echo Add curl/Postman/API test here.
                    echo.

                    echo APPLICATION TEST STAGE COMPLETED
                    echo ========================================
                '''
            }
        }


        // =========================================================
        // 10. FINAL VERIFICATION
        // =========================================================

        stage('Final Verification') {
            steps {

                echo '========================================'
                echo 'STAGE 10 - Final Verification'
                echo '========================================'

                bat '''
                    echo.
                    echo FINAL PIPELINE VERIFICATION
                    echo ========================================

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo ACE Work Directory:
                    echo %ACE_WORK_DIR%

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Checking BAR:
                    if exist "%BAR_FILE%" (
                        echo BAR EXISTS
                    ) else (
                        echo ERROR: BAR DOES NOT EXIST
                        exit /b 1
                    )

                    echo.
                    echo Checking ACE Server:
                    tasklist /FI "IMAGENAME eq IntegrationServer.exe" | find /I "IntegrationServer.exe" >nul

                    if errorlevel 1 (
                        echo ERROR: ACE Integration Server NOT RUNNING
                        exit /b 1
                    )

                    echo ACE SERVER RUNNING

                    echo.
                    echo ========================================
                    echo PIPELINE SUCCESSFUL
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
            echo 'JENKINS BUILD SUCCESSFUL'
            echo '========================================'

            echo 'GitHub checkout successful'
            echo 'ACE BAR created successfully'
            echo 'BAR deployed successfully'
            echo 'ACE Integration Server running'
            echo 'Deployment verification completed'

        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Check the failed stage and ACE logs.'

            echo 'ACE Log Directory: C:\\ACE\\servers\\TEST\\log'

        }

        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
