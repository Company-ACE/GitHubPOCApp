pipeline {

    agent any

    environment {

        // ============================================================
        // IBM ACE INSTALLATION
        // ============================================================
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'

        // ============================================================
        // ACE APPLICATION
        // ============================================================
        APP_NAME = 'GitHubPOCApp'

        // ============================================================
        // ACE SOURCE WORKSPACE CREATED DURING BUILD
        // ============================================================
        ACE_WORKSPACE = 'ace-workspace'

        // ============================================================
        // BAR FILE
        // ============================================================
        BAR_NAME = 'GitHubPOCApp.bar'
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

                /*
                 * Jenkins Declarative Pipeline automatically performs
                 * the SCM checkout before entering this stage.
                 *
                 * Therefore we do NOT call checkout scm again.
                 */

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
                '''
            }
        }


        // ============================================================
        // STAGE 2 - ACE SOURCE VALIDATION
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
                    echo ACE Version
                    echo ========================================

                    ibmint --version

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
        // STAGE 3 - PREPARE ACE APPLICATION WORKSPACE
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

                    if exist "%WORKSPACE%\\%ACE_WORKSPACE%" (
                        rmdir /S /Q "%WORKSPACE%\\%ACE_WORKSPACE%"
                    )

                    mkdir "%WORKSPACE%\\%ACE_WORKSPACE%"

                    mkdir "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%"

                    echo.
                    echo ========================================
                    echo Creating ACE Application Structure
                    echo ========================================

                    xcopy "%WORKSPACE%\\.project" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\application.descriptor" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\*.msgflow" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\*.esql" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    echo.
                    echo ========================================
                    echo ACE APPLICATION STRUCTURE
                    echo ========================================

                    dir /S /B "%WORKSPACE%\\%ACE_WORKSPACE%"

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
                    echo ACE VERSION
                    echo ========================================

                    ibmint --version

                    echo.
                    echo ========================================
                    echo Creating Build Directory
                    echo ========================================

                    if not exist "%WORKSPACE%\\builds" (
                        mkdir "%WORKSPACE%\\builds"
                    )

                    set BAR_DIR=%WORKSPACE%\\builds\\%BUILD_NUMBER%

                    mkdir "%BAR_DIR%"

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR Directory:
                    echo %BAR_DIR%

                    echo.
                    echo BAR File:
                    echo %BAR_DIR%\\%BAR_NAME%

                    echo.
                    echo ========================================
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                        --input-path "%WORKSPACE%\\%ACE_WORKSPACE%" ^
                        --output-bar-file "%BAR_DIR%\\%BAR_NAME%" ^
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

                    echo.
                    echo Generated BAR:

                    dir "%BAR_DIR%"
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
                    set BAR_DIR=%WORKSPACE%\\builds\\%BUILD_NUMBER%

                    echo.
                    echo ========================================
                    echo BAR DIRECTORY
                    echo ========================================

                    dir "%BAR_DIR%"

                    if not exist "%BAR_DIR%\\%BAR_NAME%" (
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR FILE FOUND
                    echo ========================================

                    echo.
                    echo BAR:
                    echo %BAR_DIR%\\%BAR_NAME%

                    echo.
                    echo ========================================
                    echo BAR CONTENTS
                    echo ========================================

                    jar tf "%BAR_DIR%\\%BAR_NAME%"

                    echo.
                    echo ========================================
                    echo BAR CREATED SUCCESSFULLY
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
            echo 'ACE BAR BUILD SUCCESSFUL'
            echo '========================================'

            archiveArtifacts artifacts: 'builds/**/GitHubPOCApp.bar',
                             fingerprint: true
        }


        failure {

            echo '========================================'
            echo 'ACE BAR BUILD FAILED'
            echo '========================================'
        }


        always {

            echo '========================================'
            echo 'JENKINS BUILD COMPLETED'
            echo '========================================'
        }
    }
}
