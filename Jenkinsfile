pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        APP_NAME = 'GitHubPOCApp'

        // ACE independent Integration Server work directory
        ACE_SERVER_DIR = 'C:\\ACE\\servers\\TEST'

        // Build directory
        BUILD_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"

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
                '''
            }
        }


        // ============================================================
        // STAGE 2 - ACE VALIDATION
        // ============================================================

        stage('ACE Validation') {

            steps {

                echo '========================================'
                echo 'STAGE 2 - ACE Project Validation'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo ACE Installation
                    echo ========================================
                    echo %ACE_HOME%

                    echo.
                    echo ========================================
                    echo Checking ibmint
                    echo ========================================
                    where ibmint

                    echo.
                    echo ========================================
                    echo Checking ACE Source
                    echo ========================================

                    if not exist "%WORKSPACE%\\.project" (
                        echo ERROR: .project file not found
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
                    echo ACE SOURCE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 3 - PREPARE APPLICATION
        // ============================================================

        stage('Prepare ACE Workspace') {

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
                    echo Creating ACE Application Structure
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


        // ============================================================
        // STAGE 4 - PACKAGE BAR
        // ============================================================

        stage('Package BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 4 - Package ACE BAR'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo Creating Build Directory
                    echo ========================================

                    if exist "%BUILD_DIR%" (
                        rmdir /S /Q "%BUILD_DIR%"
                    )

                    mkdir "%BUILD_DIR%"

                    echo.
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
                        --output-bar-file "%BAR_FILE%" ^
                        --project "%APP_NAME%"

                    if errorlevel 1 (
                        echo.
                        echo ========================================
                        echo ERROR: ACE BAR packaging failed
                        echo ========================================
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
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR CONTENTS
                    echo ========================================

                    jar tf "%BAR_FILE%"

                    echo.
                    echo ========================================
                    echo Checking Application ZIP
                    echo ========================================

                    jar tf "%BAR_FILE%" | findstr /I "%APP_NAME%.appzip"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: %APP_NAME%.appzip not found in BAR
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
        // STAGE 6 - PREPARE ACE SERVER
        // ============================================================

        stage('Prepare ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 6 - Prepare ACE Integration Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo ACE SERVER WORK DIRECTORY
                    echo ========================================

                    echo %ACE_SERVER_DIR%

                    if not exist "%ACE_SERVER_DIR%" (

                        echo.
                        echo ACE server work directory does not exist.
                        echo Creating server work directory...

                        mkdir "%ACE_SERVER_DIR%"

                        if errorlevel 1 (
                            echo.
                            echo ERROR: Could not create ACE server directory
                            exit /b 1
                        )

                        echo.
                        echo Creating ACE server configuration...

                        mqsicreateworkdir "%ACE_SERVER_DIR%"

                        if errorlevel 1 (
                            echo.
                            echo ERROR: mqsicreateworkdir failed
                            exit /b 1
                        )

                    ) else (

                        echo.
                        echo ACE server work directory already exists.
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER DIRECTORY
                    echo ========================================

                    dir "%ACE_SERVER_DIR%"

                    echo.
                    echo ========================================
                    echo ACE SERVER PREPARATION COMPLETED
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
                echo 'STAGE 7 - Deploy ACE BAR'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo DEPLOYMENT INFORMATION
                    echo ========================================

                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER_DIR%

                    if not exist "%BAR_FILE%" (
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    if not exist "%ACE_SERVER_DIR%" (
                        echo.
                        echo ERROR: ACE server work directory does not exist
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo DEPLOYING BAR
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --output-work-directory "%ACE_SERVER_DIR%"

                    if errorlevel 1 (
                        echo.
                        echo ========================================
                        echo ERROR: ACE BAR DEPLOYMENT FAILED
                        echo ========================================
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE BAR DEPLOYMENT SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 8 - VERIFY DEPLOYMENT
        // ============================================================

        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Verify ACE Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo ACE SERVER WORK DIRECTORY
                    echo ========================================

                    dir "%ACE_SERVER_DIR%"

                    echo.
                    echo ========================================
                    echo Searching for Application
                    echo ========================================

                    dir /S /B "%ACE_SERVER_DIR%\\%APP_NAME%*" 2>nul

                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION COMPLETED
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 9 - START ACE SERVER
        // ============================================================

        stage('Start ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo Starting ACE Server
                    echo ========================================

                    echo Server Work Directory:
                    echo %ACE_SERVER_DIR%

                    echo.
                    echo NOTE:
                    echo The Jenkins process will start the ACE
                    echo Integration Server in the background.
                    echo.

                    start "ACE-TEST-SERVER" /B ^
                        "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" ^
                        --work-dir "%ACE_SERVER_DIR%" ^
                        > "%ACE_SERVER_DIR%\\jenkins-start.log" 2>&1

                    timeout /T 15 /NOBREAK

                    echo.
                    echo ========================================
                    echo ACE SERVER START COMMAND COMPLETED
                    echo ========================================

                    if exist "%ACE_SERVER_DIR%\\jenkins-start.log" (
                        echo.
                        echo ========================================
                        echo ACE STARTUP LOG
                        echo ========================================
                        type "%ACE_SERVER_DIR%\\jenkins-start.log"
                    )
                '''
            }
        }


        // ============================================================
        // STAGE 10 - FINAL CHECK
        // ============================================================

        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 10 - Final Verification'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo FINAL ACE DEPLOYMENT CHECK
                    echo ========================================

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo SERVER:
                    echo %ACE_SERVER_DIR%

                    echo.
                    echo Application:
                    dir /S /B "%ACE_SERVER_DIR%\\%APP_NAME%*" 2>nul

                    echo.
                    echo ========================================
                    echo ACE CI/CD PIPELINE COMPLETED
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

            echo 'ACE BAR successfully packaged and deployed.'

            archiveArtifacts artifacts: 'builds/**/*.bar',
                             fingerprint: true
        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Please check the failed stage in Jenkins console.'
        }

        always {

            echo '========================================'
            echo 'PIPELINE COMPLETED'
            echo '========================================'
        }
    }
}
