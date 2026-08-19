pipeline {

    agent any

    environment {

        // ============================================================
        // ACE INSTALLATION
        // ============================================================
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        // ============================================================
        // ACE INDEPENDENT INTEGRATION SERVER
        // ============================================================
        ACE_SERVER = 'C:\\ACE\\servers\\TEST'

        // ============================================================
        // APPLICATION
        // ============================================================
        APP_NAME = 'GitHubPOCApp'

        // ============================================================
        // BAR DIRECTORY
        // ============================================================
        BAR_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"
        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"

        // ============================================================
        // ACE WORKSPACE
        // ============================================================
        ACE_WORKSPACE = "${WORKSPACE}\\ace-workspace"

    }

    stages {

        // ============================================================
        // 1. CHECKOUT
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
        // 2. VALIDATE ACE
        // ============================================================
        stage('ACE Validation') {

            steps {

                echo '========================================'
                echo 'STAGE 2 - ACE Validation'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo ACE Installation
                    echo ========================================
                    echo %ACE_HOME%

                    echo.
                    echo Checking ibmint
                    where ibmint

                    if errorlevel 1 (
                        echo ERROR: ibmint not found
                        exit /b 1
                    )

                    echo.
                    echo ACE VERSION
                    ibmint version

                    echo.
                    echo ACE SOURCE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // 3. PREPARE ACE APPLICATION
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

                    if exist "%ACE_WORKSPACE%" (
                        rmdir /S /Q "%ACE_WORKSPACE%"
                    )

                    mkdir "%ACE_WORKSPACE%"
                    mkdir "%ACE_WORKSPACE%\\%APP_NAME%"

                    echo.
                    echo ========================================
                    echo Copying ACE Application Files
                    echo ========================================

                    copy /Y "%WORKSPACE%\\.project" ^
                        "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" ^
                        "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" ^
                        "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" ^
                        "%ACE_WORKSPACE%\\%APP_NAME%\\"

                    echo.
                    echo ========================================
                    echo ACE APPLICATION STRUCTURE
                    echo ========================================

                    dir /S /B "%ACE_WORKSPACE%"

                    echo.
                    echo ACE WORKSPACE PREPARATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // 4. PACKAGE BAR
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
                    echo ========================================
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                        --input-path "%ACE_WORKSPACE%" ^
                        --output-bar-file "%BAR_FILE%"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: BAR PACKAGING FAILED
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
        // 5. VERIFY BAR
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
        // 6. CHECK ACE SERVER
        // ============================================================
        stage('Check ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 6 - Check ACE Integration Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo ACE SERVER WORK DIRECTORY
                    echo ========================================

                    echo %ACE_SERVER%

                    if not exist "%ACE_SERVER%" (
                        echo.
                        echo ERROR: ACE Server does not exist
                        echo.
                        echo Please create the ACE server manually first.
                        echo.
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER DIRECTORY FOUND
                    echo ========================================

                    dir "%ACE_SERVER%"

                    if not exist "%ACE_SERVER%\\server.conf.yaml" (
                        echo.
                        echo ERROR: server.conf.yaml not found
                        exit /b 1
                    )

                    echo.
                    echo ACE SERVER VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // 7. DEPLOY BAR
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
                    echo ACE SERVER:
                    echo %ACE_SERVER%

                    echo.
                    echo ========================================
                    echo Deploying BAR
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --output-work-directory "%ACE_SERVER%" ^
                        --restart-all-applications

                    if errorlevel 1 (
                        echo.
                        echo ========================================
                        echo ERROR: BAR DEPLOYMENT FAILED
                        echo ========================================
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
        // 8. VERIFY DEPLOYMENT
        // ============================================================
        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 8 - Verify Deployment'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo VERIFYING DEPLOYMENT
                    echo ========================================

                    echo.
                    echo Server:
                    echo %ACE_SERVER%

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo ========================================
                    echo Checking deployed files
                    echo ========================================

                    dir "%ACE_SERVER%\\run" /S /B

                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION COMPLETED
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // 9. START ACE SERVER
        // ============================================================
        stage('Start ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Start ACE Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo Starting ACE Server
                    echo ========================================

                    ibman start "%ACE_SERVER%"

                    if errorlevel 1 (
                        echo.
                        echo WARNING: ibman start returned an error.
                        echo Server may already be running.
                    )

                    echo.
                    echo Waiting for ACE server...
                    timeout /T 10 /NOBREAK

                    echo.
                    echo ACE SERVER START COMMAND COMPLETED
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // 10. FINAL VERIFICATION
        // ============================================================
        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 10 - Final Verification'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo FINAL ACE DEPLOYMENT CHECK
                    echo ========================================

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER%

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo ========================================
                    echo Pipeline Deployment Completed
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
            echo "ACE Server: ${ACE_SERVER}"

        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Please check the failed stage in Jenkins console.'
        }

        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
