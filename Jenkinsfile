pipeline {

    agent any

    environment {
        ACE_HOME       = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_PROFILE    = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'
        ACE_SERVER     = 'C:\\ACE\\servers\\TEST'
        APP_NAME       = 'GitHubPOCApp'
        BAR_NAME       = 'GitHubPOCApp.bar'
        REPOSITORY     = 'https://github.com/Company-ACE/GitHubPOCApp.git'
        BRANCH         = 'dev'
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
                    call "%ACE_PROFILE%"
                    if errorlevel 1 exit /b 1

                    echo.
                    echo ========================================
                    echo ACE Installation
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
                    echo Checking ibmint.exe:
                    echo %ACE_HOME%\\server\\bin\\ibmint.exe

                    echo.
                    echo ACE Version:
                    "%ACE_HOME%\\server\\bin\\ibmint.exe" --version

                    if errorlevel 1 (
                        echo WARNING: ibmint --version returned non-zero
                        echo Continuing because ACE installation is present
                    )

                    echo.
                    echo ========================================
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================

                    exit /b 0
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

                    exit /b 0
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
                    call "%ACE_PROFILE%"
                    if errorlevel 1 exit /b 1

                    echo.
                    echo ========================================
                    echo Creating BAR Directory
                    echo ========================================

                    echo Build Number:
                    echo %BUILD_NUMBER%

                    if not exist "%WORKSPACE%\\builds\\%BUILD_NUMBER%" (
                        mkdir "%WORKSPACE%\\builds\\%BUILD_NUMBER%"
                    )

                    set BAR_DIR=%WORKSPACE%\\builds\\%BUILD_NUMBER%
                    set BAR_FILE=%BAR_DIR%\\%BAR_NAME%

                    echo.
                    echo BAR Directory:
                    echo %BAR_DIR%

                    echo.
                    echo BAR File:
                    echo %BAR_FILE%

                    echo.
                    echo ========================================
                    echo Running ibmint package
                    echo ========================================

                    "%ACE_HOME%\\server\\bin\\ibmint.exe" package ^
                        --input-path "%WORKSPACE%\\ace-workspace\\%APP_NAME%" ^
                        --output-bar-file "%BAR_FILE%"

                    if errorlevel 1 (
                        echo ERROR: BAR packaging failed
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR PACKAGING COMPLETED
                    echo ========================================

                    dir "%BAR_DIR%"

                    exit /b 0
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
                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

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

                    exit /b 0
                '''
            }
        }


        // =========================================================
        // STAGE 6
        // CREATE ACE SERVER IF IT DOES NOT EXIST
        // =========================================================
        stage('Check/Create ACE Server') {
            steps {

                echo '========================================'
                echo 'STAGE 6 - Check/Create ACE Integration Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"
                    if errorlevel 1 exit /b 1

                    echo.
                    echo ========================================
                    echo ACE SERVER WORK DIRECTORY
                    echo ========================================

                    echo %ACE_SERVER%

                    echo.
                    echo ========================================
                    echo CHECKING ACE SERVER
                    echo ========================================

                    if exist "%ACE_SERVER%" (

                        echo ACE Server directory already exists.
                        echo.

                        if exist "%ACE_SERVER%\\server.conf.yaml" (
                            echo server.conf.yaml found.
                            echo ACE Server already configured.
                        ) else (
                            echo server.conf.yaml not found.
                            echo Creating ACE Server...
                            
                            "%ACE_HOME%\\server\\bin\\ibmint.exe" create server ^
                                --name TEST ^
                                --server-work-dir "%ACE_SERVER%"

                            if errorlevel 1 (
                                echo ERROR: Failed to create ACE Server
                                exit /b 1
                            )
                        )

                    ) else (

                        echo ACE Server does not exist.
                        echo.
                        echo Creating ACE Server:
                        echo %ACE_SERVER%

                        "%ACE_HOME%\\server\\bin\\ibmint.exe" create server ^
                            --name TEST ^
                            --server-work-dir "%ACE_SERVER%"

                        if errorlevel 1 (
                            echo ERROR: Failed to create ACE Server
                            exit /b 1
                        )

                    )

                    echo.
                    echo ========================================
                    echo VERIFYING ACE SERVER
                    echo ========================================

                    if not exist "%ACE_SERVER%" (
                        echo ERROR: ACE Server directory was not created
                        exit /b 1
                    )

                    if not exist "%ACE_SERVER%\\server.conf.yaml" (
                        echo ERROR: server.conf.yaml was not created
                        exit /b 1
                    )

                    echo.
                    echo ACE SERVER DIRECTORY:
                    dir "%ACE_SERVER%"

                    echo.
                    echo ========================================
                    echo ACE SERVER READY
                    echo ========================================

                    exit /b 0
                '''
            }
        }


        // =========================================================
        // STAGE 7
        // =========================================================
        stage('Deploy BAR') {
            steps {

                echo '========================================'
                echo 'STAGE 7 - Deploy ACE BAR'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"
                    if errorlevel 1 exit /b 1

                    set BAR_FILE=%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

                    echo.
                    echo ========================================
                    echo DEPLOYMENT INFORMATION
                    echo ========================================

                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER%

                    echo.
                    echo ========================================
                    echo DEPLOYING BAR
                    echo ========================================

                    "%ACE_HOME%\\server\\bin\\ibmint.exe" deploy ^
                        --input-file "%BAR_FILE%" ^
                        --work-dir "%ACE_SERVER%"

                    if errorlevel 1 (
                        echo ERROR: BAR deployment failed
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR DEPLOYMENT SUCCESSFUL
                    echo ========================================

                    exit /b 0
                '''
            }
        }


        // =========================================================
        // STAGE 8
        // =========================================================
        stage('Verify Deployment') {
            steps {

                echo '========================================'
                echo 'STAGE 8 - Verify Deployment'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"
                    if errorlevel 1 exit /b 1

                    echo.
                    echo ========================================
                    echo VERIFYING ACE DEPLOYMENT
                    echo ========================================

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER%

                    echo.
                    echo ========================================
                    echo Checking runtime directory
                    echo ========================================

                    if not exist "%ACE_SERVER%\\run\\%APP_NAME%" (
                        echo ERROR: Application runtime directory not found
                        exit /b 1
                    )

                    dir /S /B "%ACE_SERVER%\\run\\%APP_NAME%"

                    echo.
                    echo ========================================
                    echo APPLICATION DEPLOYED
                    echo ========================================

                    exit /b 0
                '''
            }
        }


        // =========================================================
        // STAGE 9
        // =========================================================
        stage('Final Verification') {
            steps {

                echo '========================================'
                echo 'STAGE 9 - Final Verification'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo FINAL DEPLOYMENT STATUS
                    echo ========================================

                    echo.
                    echo GitHub Repository:
                    echo %REPOSITORY%

                    echo.
                    echo Branch:
                    echo %BRANCH%

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Build:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%BAR_NAME%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER%

                    echo.
                    echo ========================================
                    echo ACE DEPLOYMENT PIPELINE COMPLETED
                    echo ========================================

                    exit /b 0
                '''
            }
        }
    }


    // =============================================================
    // POST ACTIONS
    // =============================================================
    post {

        success {

            echo '========================================'
            echo 'JENKINS BUILD SUCCESSFUL'
            echo '========================================'

            echo "Application : ${APP_NAME}"
            echo "Build       : ${BUILD_NUMBER}"
            echo "BAR         : ${WORKSPACE}\\builds\\${BUILD_NUMBER}\\${BAR_NAME}"
            echo "ACE Server  : ${ACE_SERVER}"

            echo '========================================'
            echo 'GITHUB -> JENKINS -> ACE DEPLOYMENT OK'
            echo '========================================'
        }

        failure {

            echo '========================================'
            echo 'JENKINS BUILD FAILED'
            echo '========================================'

            echo 'Check the failed stage in the Jenkins console.'

            echo '========================================'
        }

        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
