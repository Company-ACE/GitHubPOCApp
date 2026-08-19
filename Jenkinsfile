pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_SERVER_DIR = 'C:\\ACE\\servers\\TEST'
        ACE_SERVER_NAME = 'TEST'
        APP_NAME = 'GitHubPOCApp'
        BUILD_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"
        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"
    }

    stages {

        /*
         * ============================================================
         * STAGE 1 - CHECKOUT
         * ============================================================
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
                echo Source Files:
                dir

                echo.
                echo ========================================
                echo GITHUB CHECKOUT SUCCESSFUL
                echo ========================================
                '''
            }
        }


        /*
         * ============================================================
         * STAGE 2 - ACE VALIDATION
         * ============================================================
         */
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
                where ibmint

                if not exist "%ACE_HOME%\\server\\bin\\ibmint.exe" (
                    echo ERROR: ibmint.exe not found
                    exit /b 1
                )

                echo.
                echo ACE Version:
                ibmint --version

                echo.
                echo ========================================
                echo ACE VALIDATION SUCCESSFUL
                echo ========================================
                '''
            }
        }


        /*
         * ============================================================
         * STAGE 3 - PREPARE ACE APPLICATION
         * ============================================================
         */
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
                echo Copying ACE Application Files
                echo ========================================

                if not exist "%WORKSPACE%\\.project" (
                    echo ERROR: .project not found
                    exit /b 1
                )

                if not exist "%WORKSPACE%\\application.descriptor" (
                    echo ERROR: application.descriptor not found
                    exit /b 1
                )

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


        /*
         * ============================================================
         * STAGE 4 - PACKAGE BAR
         * ============================================================
         */
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

                echo Build Number:
                echo %BUILD_NUMBER%

                echo.
                echo BAR Directory:
                echo %BUILD_DIR%

                echo.
                echo BAR File:
                echo %BAR_FILE%

                if not exist "%BUILD_DIR%" (
                    mkdir "%BUILD_DIR%"
                )

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


        /*
         * ============================================================
         * STAGE 5 - VERIFY BAR
         * ============================================================
         */
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


        /*
         * ============================================================
         * STAGE 6 - CREATE / START ACE SERVER
         *
         * IMPORTANT:
         * This is a STANDALONE ACE Integration Server.
         *
         * We do NOT use:
         *
         * ibmint create server --name
         *
         * because ACE 12.0.12.22 does not support --name.
         *
         * The IntegrationServer command initializes the work
         * directory when it starts.
         * ============================================================
         */
        stage('Check/Create ACE Server') {
            steps {

                echo '========================================'
                echo 'STAGE 6 - Check/Create ACE Integration Server'
                echo '========================================'

                bat '''
                call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                echo.
                echo ========================================
                echo ACE SERVER CONFIGURATION
                echo ========================================

                echo Server Name:
                echo %ACE_SERVER_NAME%

                echo Server Directory:
                echo %ACE_SERVER_DIR%

                echo.

                if not exist "C:\\ACE" (
                    echo Creating C:\\ACE
                    mkdir "C:\\ACE"
                )

                if not exist "C:\\ACE\\servers" (
                    echo Creating C:\\ACE\\servers
                    mkdir "C:\\ACE\\servers"
                )

                if not exist "%ACE_SERVER_DIR%" (

                    echo.
                    echo ========================================
                    echo ACE SERVER DOES NOT EXIST
                    echo ========================================

                    echo Creating server work directory:
                    echo %ACE_SERVER_DIR%

                    mkdir "%ACE_SERVER_DIR%"

                    if errorlevel 1 (
                        echo ERROR: Unable to create server directory
                        exit /b 1
                    )

                    echo.
                    echo Server directory created successfully.

                ) else (

                    echo.
                    echo ========================================
                    echo ACE SERVER DIRECTORY ALREADY EXISTS
                    echo ========================================

                    echo %ACE_SERVER_DIR%
                )

                echo.
                echo ========================================
                echo CHECKING ACE SERVER
                echo ========================================

                if not exist "%ACE_SERVER_DIR%" (
                    echo ERROR: ACE Server directory does not exist
                    exit /b 1
                )

                echo.
                echo Starting ACE Integration Server...

                start "ACE-TEST-SERVER" /B ^
                    "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" ^
                    -w "%ACE_SERVER_DIR%"

                echo.
                echo Waiting for ACE server to initialize...

                timeout /t 15 /nobreak

                echo.
                echo ========================================
                echo ACE SERVER START COMMAND COMPLETED
                echo ========================================

                dir "%ACE_SERVER_DIR%"

                '''
            }
        }


        /*
         * ============================================================
         * STAGE 7 - DEPLOY BAR
         * ============================================================
         */
        stage('Deploy BAR') {
            steps {

                echo '========================================'
                echo 'STAGE 7 - Deploy BAR'
                echo '========================================'

                bat '''
                call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                echo.
                echo ========================================
                echo DEPLOYING BAR
                echo ========================================

                echo BAR:
                echo %BAR_FILE%

                echo.
                echo Server Work Directory:
                echo %ACE_SERVER_DIR%

                if not exist "%BAR_FILE%" (
                    echo ERROR: BAR file not found
                    exit /b 1
                )

                if not exist "%ACE_SERVER_DIR%" (
                    echo ERROR: ACE server directory not found
                    exit /b 1
                )

                echo.
                echo Running ibmint deploy...

                ibmint deploy ^
                    --input-bar-file "%BAR_FILE%" ^
                    --work-dir "%ACE_SERVER_DIR%"

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


        /*
         * ============================================================
         * STAGE 8 - VERIFY DEPLOYMENT
         * ============================================================
         */
        stage('Verify Deployment') {
            steps {

                echo '========================================'
                echo 'STAGE 8 - Verify Deployment'
                echo '========================================'

                bat '''
                echo.
                echo ========================================
                echo CHECKING DEPLOYED APPLICATION
                echo ========================================

                echo Server Work Directory:
                echo %ACE_SERVER_DIR%

                echo.
                echo Looking for application:
                echo %APP_NAME%

                if not exist "%ACE_SERVER_DIR%" (
                    echo ERROR: Server work directory does not exist
                    exit /b 1
                )

                echo.
                echo Server directory:
                dir /S /B "%ACE_SERVER_DIR%"

                echo.
                echo ========================================
                echo DEPLOYMENT VERIFICATION COMPLETED
                echo ========================================
                '''
            }
        }


        /*
         * ============================================================
         * STAGE 9 - FINAL VERIFICATION
         * ============================================================
         */
        stage('Final Verification') {
            steps {

                echo '========================================'
                echo 'STAGE 9 - Final Verification'
                echo '========================================'

                bat '''
                call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                echo.
                echo ========================================
                echo FINAL ACE VERIFICATION
                echo ========================================

                echo ACE Version:
                ibmint --version

                echo.
                echo Server Directory:
                echo %ACE_SERVER_DIR%

                echo.
                echo Checking IntegrationServer process:

                tasklist | findstr /I "IntegrationServer"

                if errorlevel 1 (
                    echo WARNING: IntegrationServer process not found
                ) else (
                    echo IntegrationServer process is running
                )

                echo.
                echo ========================================
                echo FINAL VERIFICATION COMPLETED
                echo ========================================
                '''
            }
        }
    }


    /*
     * ================================================================
     * POST ACTIONS
     * ================================================================
     */
    post {

        success {

            echo '========================================'
            echo 'PIPELINE FINISHED SUCCESSFULLY'
            echo '========================================'

            echo 'GitHub -> Jenkins -> BAR -> ACE Deployment SUCCESS'

            echo '========================================'
        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Check the failed stage in the Jenkins console.'

            echo '========================================'
        }
    }
}
