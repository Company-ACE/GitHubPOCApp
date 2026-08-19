pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        APP_NAME = 'GitHubPOCApp'
        BUILD_DIR = 'build'
        BAR_NAME = 'GitHubPOCApp.bar'
        ACE_WORKSPACE = 'ace-workspace'
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

                // Jenkins already performs Declarative Checkout SCM.
                // Do not checkout again here.

                bat '''
                    echo.
                    echo Jenkins Workspace:
                    echo %WORKSPACE%

                    echo.
                    echo Git Commit:
                    git rev-parse HEAD

                    echo.
                    echo Source files:
                    dir
                '''
            }
        }


        // ============================================================
        // 2. VALIDATE ACE SOURCE
        // ============================================================
        stage('ACE Validation') {
            steps {

                echo '========================================'
                echo 'STAGE 2 - ACE Project Validation'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo ACE Installation:
                    echo %ACE_HOME%

                    echo.
                    echo Checking ibmint:
                    where ibmint

                    echo.
                    echo Checking ACE project:

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
                    echo ACE source files validated successfully.
                '''
            }
        }


        // ============================================================
        // 3. CREATE ACE APPLICATION WORKSPACE
        // ============================================================
        stage('Prepare ACE Workspace') {
            steps {

                echo '========================================'
                echo 'STAGE 3 - Prepare ACE Application'
                echo '========================================'

                bat '''
                    echo.
                    echo Cleaning previous build directories...

                    if exist "%WORKSPACE%\\%ACE_WORKSPACE%" (
                        rmdir /S /Q "%WORKSPACE%\\%ACE_WORKSPACE%"
                    )

                    if exist "%WORKSPACE%\\%BUILD_DIR%" (
                        rmdir /S /Q "%WORKSPACE%\\%BUILD_DIR%"
                    )

                    mkdir "%WORKSPACE%\\%ACE_WORKSPACE%"
                    mkdir "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%"
                    mkdir "%WORKSPACE%\\%BUILD_DIR%"

                    echo.
                    echo Creating ACE application structure...

                    xcopy "%WORKSPACE%\\.project" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\application.descriptor" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\*.msgflow" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    xcopy "%WORKSPACE%\\*.esql" ^
                          "%WORKSPACE%\\%ACE_WORKSPACE%\\%APP_NAME%\\" /Y

                    echo.
                    echo ACE Application structure:

                    dir /S /B "%WORKSPACE%\\%ACE_WORKSPACE%"
                '''
            }
        }


        // ============================================================
        // 4. PACKAGE BAR USING IBM ACE ibmint
        // ============================================================
        stage('Package BAR') {
            steps {

                echo '========================================'
                echo 'STAGE 4 - Package ACE BAR'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo Checking ibmint:
                    where ibmint

                    echo.
                    echo Creating BAR:

                    echo BAR:
                    echo %WORKSPACE%\\%BUILD_DIR%\\%BAR_NAME%

                    echo.
                    echo Running ibmint package...

                    ibmint package ^
                        --input-path "%WORKSPACE%\\%ACE_WORKSPACE%" ^
                        --output-bar-file "%WORKSPACE%\\%BUILD_DIR%\\%BAR_NAME%" ^
                        --application "%APP_NAME%"

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
                    echo BAR directory:

                    dir "%WORKSPACE%\\%BUILD_DIR%"

                    if not exist "%WORKSPACE%\\%BUILD_DIR%\\%BAR_NAME%" (
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR CREATED SUCCESSFULLY
                    echo ========================================

                    echo.
                    echo BAR contents:

                    jar tf "%WORKSPACE%\\%BUILD_DIR%\\%BAR_NAME%"
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

            archiveArtifacts artifacts: 'build/*.bar',
                             fingerprint: true
        }

        failure {

            echo '========================================'
            echo 'ACE BAR BUILD FAILED'
            echo '========================================'
        }

        always {

            echo '========================================'
            echo 'BUILD COMPLETED'
            echo '========================================'
        }
    }
}
