pipeline {

    agent any

    environment {

        // ============================================================
        // ACE INSTALLATION
        // ============================================================
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        // ============================================================
        // ACE APPLICATION
        // ============================================================
        APP_NAME = 'GitHubPOCApp'
        BAR_NAME = 'GitHubPOCApp.bar'

        // ============================================================
        // ACE INDEPENDENT SERVER
        // CHANGE THIS IF YOUR SERVER DIRECTORY IS DIFFERENT
        // ============================================================
        ACE_SERVER_WORKDIR = 'C:\\ACE\\servers\\TEST'
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
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

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
                        echo ERROR: .project not found
                        exit /b 1
                    )

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


        // ============================================================
        // STAGE 3 - PREPARE ACE APPLICATION
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

                    xcopy "%WORKSPACE%\\.project" ^
                          "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\application.descriptor" ^
                          "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\*.msgflow" ^
                          "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\*.esql" ^
                          "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y

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
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo ========================================
                    echo Creating Build Directory
                    echo ========================================

                    if exist "%WORKSPACE%\\builds\\%BUILD_NUMBER%" (
                        rmdir /S /Q "%WORKSPACE%\\builds\\%BUILD_NUMBER%"
                    )

                    mkdir "%WORKSPACE%\\builds\\%BUILD_NUMBER%"

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
                    echo ========================================
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                        --input-path "%WORKSPACE%\\ace-workspace" ^
                        --output-bar-file "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%" ^
                        --project "%APP_NAME%"

                    if errorlevel 1 (
                        echo.
                        echo ========================================
                        echo ERROR: ACE BAR PACKAGING FAILED
                        echo ========================================
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR PACKAGING COMPLETED
                    echo ========================================

                    dir "%WORKSPACE%\\builds\\%BUILD_NUMBER%"
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
                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

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
        // STAGE 6 - DEPLOY BAR
        // ============================================================
        stage('Deploy BAR') {

            steps {

                echo '========================================'
                echo 'STAGE 6 - Deploy ACE BAR'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

                    echo.
                    echo ========================================
                    echo Deployment Information
                    echo ========================================

                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER_WORKDIR%

                    if not exist "%BAR_FILE%" (
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    if not exist "%ACE_SERVER_WORKDIR%" (
                        echo.
                        echo ERROR: ACE Server work directory does not exist
                        echo %ACE_SERVER_WORKDIR%
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo Deploying BAR
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --work-dir "%ACE_SERVER_WORKDIR%"

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
        // STAGE 7 - DEPLOYMENT VERIFICATION
        // ============================================================
        stage('Verify Deployment') {

            steps {

                echo '========================================'
                echo 'STAGE 7 - Verify ACE Deployment'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo ========================================
                    echo Checking ACE Server
                    echo ========================================

                    echo Server Work Directory:
                    echo %ACE_SERVER_WORKDIR%

                    echo.
                    echo ========================================
                    echo Deployment Completed
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Build:
                    echo %BUILD_NUMBER%

                    echo.
                    echo ========================================
                    echo ACE DEPLOYMENT COMPLETED
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

            echo 'Application: GitHubPOCApp'
            echo 'Build Number: ${BUILD_NUMBER}'
            echo 'BAR: builds/${BUILD_NUMBER}/GitHubPOCApp.bar'

            archiveArtifacts artifacts: 'builds/*/*.bar',
                             fingerprint: true
        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Please check the failed stage in Jenkins console.'
        }
    }
}
