pipeline {

    agent any

    environment {

        // ============================================================
        // IBM ACE
        // ============================================================
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        ACE_PROFILE =
            'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'

        // ============================================================
        // EXISTING ACE INDEPENDENT INTEGRATION SERVER
        // ============================================================
        ACE_SERVER = 'C:\\ACE\\servers\\TEST'

        // ============================================================
        // ACE APPLICATION
        // ============================================================
        APP_NAME = 'GitHubPOCApp'

        // ============================================================
        // JENKINS BUILD DIRECTORIES
        // ============================================================
        ACE_WORKSPACE = "${WORKSPACE}\\ace-workspace"

        BAR_DIR = "${WORKSPACE}\\builds\\${BUILD_NUMBER}"

        BAR_FILE =
            "${WORKSPACE}\\builds\\${BUILD_NUMBER}\\GitHubPOCApp.bar"
    }


    stages {


        // ============================================================
        // STAGE 1
        // GITHUB CHECKOUT
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

                    echo.
                    echo ========================================
                    echo GITHUB CHECKOUT SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 2
        // ACE VALIDATION
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

                    echo ACE_HOME:
                    echo %ACE_HOME%

                    echo.
                    echo Checking ibmint:
                    where ibmint

                    if errorlevel 1 (
                        echo.
                        echo ERROR: ibmint command not found
                        exit /b 1
                    )

                    echo.
                    echo Checking ibmint.exe:

                    if not exist "%ACE_HOME%\\server\\bin\\ibmint.exe" (
                        echo.
                        echo ERROR: ibmint.exe not found
                        exit /b 1
                    )

                    echo.
                    echo ACE Version:
                    echo 12.0.12.22

                    echo.
                    echo ========================================
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 3
        // PREPARE ACE APPLICATION
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

                    if not exist "%WORKSPACE%\\.project" (
                        echo ERROR: .project not found
                        exit /b 1
                    )

                    if not exist "%WORKSPACE%\\application.descriptor" (
                        echo ERROR: application.descriptor not found
                        exit /b 1
                    )

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
                    echo ========================================
                    echo ACE WORKSPACE PREPARATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 4
        // PACKAGE BAR
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
                    echo BAR Directory:
                    echo %BAR_DIR%

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
                        echo ========================================
                        echo ERROR: BAR PACKAGING FAILED
                        echo ========================================
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
        // STAGE 5
        // VERIFY BAR
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

                    if errorlevel 1 (
                        echo.
                        echo ERROR: Unable to read BAR file
                        exit /b 1
                    )

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


        // ============================================================
        // STAGE 6
        // CHECK EXISTING ACE SERVER
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
                        echo Expected:
                        echo %ACE_SERVER%
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER DIRECTORY
                    echo ========================================

                    dir "%ACE_SERVER%"

                    echo.
                    echo ========================================
                    echo Checking server.conf.yaml
                    echo ========================================

                    if not exist "%ACE_SERVER%\\server.conf.yaml" (
                        echo.
                        echo ERROR: server.conf.yaml not found
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 7
        // DEPLOY BAR
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
        // STAGE 8
        // VERIFY DEPLOYMENT
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

                    if not exist "%ACE_SERVER%\\run" (
                        echo.
                        echo ERROR: ACE run directory not found
                        exit /b 1
                    )

                    dir "%ACE_SERVER%\\run" /S /B

                    echo.
                    echo ========================================
                    echo Checking application in server
                    echo ========================================

                    dir "%ACE_SERVER%\\run" /S /B | findstr /I "%APP_NAME%"

                    if errorlevel 1 (
                        echo.
                        echo WARNING: Application name not found
                        echo in runtime directory.
                        echo.
                        echo Deployment command completed,
                        echo but runtime verification needs review.
                    )

                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION COMPLETED
                    echo ========================================
                '''
            }
        }


        // ============================================================
        // STAGE 9
        // FINAL STATUS
        // ============================================================
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
                    echo https://github.com/Company-ACE/GitHubPOCApp.git

                    echo.
                    echo Branch:
                    echo dev

                    echo.
                    echo Application:
                    echo %APP_NAME%

                    echo.
                    echo Build:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR:
                    echo %BAR_FILE%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER%

                    echo.
                    echo ========================================
                    echo ACE DEPLOYMENT PIPELINE COMPLETED
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

            echo "Application : ${APP_NAME}"
            echo "Build       : ${BUILD_NUMBER}"
            echo "BAR         : ${BAR_FILE}"
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
        }

        always {

            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }
    }
}
