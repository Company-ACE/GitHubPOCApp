pipeline {

    agent any

    environment {

        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_PROFILE = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        APP_NAME = 'GitHubPOCApp'

        ACE_SERVER_DIR = 'C:\\ACE\\servers\\TEST'

        BUILD_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"
        BAR_FILE = "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"

        ACE_SERVER_NAME = 'TEST'
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
                    echo Checking ibmint
                    echo ========================================
                    where ibmint

                    if not exist "%ACE_HOME%\\server\\bin\\ibmint.exe" (
                        echo ERROR: ibmint.exe not found
                        exit /b 1
                    )

                    echo.
                    echo Checking ACE Source
                    echo ========================================

                    if not exist "%WORKSPACE%\\application.descriptor" (
                        echo ERROR: application.descriptor not found
                        exit /b 1
                    )

                    if not exist "%WORKSPACE%\\GitHubPOCFlow.msgflow" (
                        echo ERROR: GitHubPOCFlow.msgflow not found
                        exit /b 1
                    )

                    if not exist "%WORKSPACE%\\GitHubPOCFlow_Compute.esql" (
                        echo ERROR: GitHubPOCFlow_Compute.esql not found
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SOURCE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        /*
         * ============================================================
         * STAGE 3 - PREPARE ACE APPLICATION
         * ============================================================
         */

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
                        --output-bar-file "%BAR_FILE%"

                    if errorlevel 1 (
                        echo.
                        echo ========================================
                        echo ERROR: BAR packaging failed
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
         * STAGE 6 - CHECK EXISTING ACE SERVER
         * ============================================================
         */

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

                    echo %ACE_SERVER_DIR%

                    if not exist "%ACE_SERVER_DIR%" (
                        echo.
                        echo ========================================
                        echo ERROR
                        echo ========================================
                        echo ACE server work directory does not exist:
                        echo %ACE_SERVER_DIR%
                        echo.
                        echo Create the ACE server manually first.
                        echo Jenkins will NOT create the server.
                        echo ========================================
                        exit /b 1
                    )

                    if not exist "%ACE_SERVER_DIR%\\server.conf.yaml" (
                        echo.
                        echo ERROR: server.conf.yaml not found
                        echo %ACE_SERVER_DIR%
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER DIRECTORY FOUND
                    echo ========================================

                    dir "%ACE_SERVER_DIR%"

                    echo.
                    echo ========================================
                    echo ACE SERVER VALIDATION SUCCESSFUL
                    echo ========================================
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
                    echo %ACE_SERVER_DIR%

                    echo.
                    echo ========================================
                    echo Deploying BAR
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --work-directory "%ACE_SERVER_DIR%"

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
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo DEPLOYED APPLICATIONS
                    echo ========================================

                    ibmint get applications ^
                        --work-directory "%ACE_SERVER_DIR%"

                    if errorlevel 1 (
                        echo.
                        echo WARNING: Could not retrieve application list
                    )

                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION COMPLETED
                    echo ========================================
                '''
            }
        }


        /*
         * ============================================================
         * STAGE 9 - START SERVER
         * ============================================================
         */

        stage('Start ACE Server') {

            steps {

                echo '========================================'
                echo 'STAGE 9 - Start ACE Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo Checking ACE Server
                    echo ========================================

                    ibmint get server ^
                        --work-directory "%ACE_SERVER_DIR%"

                    echo.
                    echo ========================================
                    echo Starting ACE Server
                    echo ========================================

                    ibmint start server ^
                        --work-directory "%ACE_SERVER_DIR%"

                    if errorlevel 1 (
                        echo.
                        echo WARNING: Server start command returned an error.
                        echo Server may already be running.
                    )

                    timeout /t 10 /nobreak
                '''
            }
        }


        /*
         * ============================================================
         * STAGE 10 - FINAL VERIFICATION
         * ============================================================
         */

        stage('Final Verification') {

            steps {

                echo '========================================'
                echo 'STAGE 10 - Final Verification'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo FINAL ACE SERVER STATUS
                    echo ========================================

                    ibmint get server ^
                        --work-directory "%ACE_SERVER_DIR%"

                    echo.
                    echo ========================================
                    echo DEPLOYED APPLICATION
                    echo ========================================

                    ibmint get applications ^
                        --work-directory "%ACE_SERVER_DIR%"

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
            echo 'JENKINS BUILD SUCCESSFUL'
            echo '========================================'

            echo 'ACE BAR created and deployment completed.'

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
