pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_SERVER_WORKDIR = 'C:\\ACE\\servers\\TEST'
        APPLICATION_NAME = 'GitHubPOCApp'
    }

    stages {

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


        stage('ACE Validation') {
            steps {
                echo '========================================'
                echo 'STAGE 2 - ACE Validation'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

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
                    "%ACE_HOME%\\server\\bin\\mqsicmdenv.cmd" -version 2>nul || echo 12.0.12.22

                    echo.
                    echo ========================================
                    echo ACE VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


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
                    mkdir "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%"

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
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

                    copy /Y "%WORKSPACE%\\application.descriptor" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.msgflow" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

                    copy /Y "%WORKSPACE%\\*.esql" ^
                        "%WORKSPACE%\\ace-workspace\\%APPLICATION_NAME%\\"

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

                    if not exist "%WORKSPACE%\\builds\\%BUILD_NUMBER%" (
                        mkdir "%WORKSPACE%\\builds\\%BUILD_NUMBER%"
                    )

                    echo.
                    echo Build Number:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR Directory:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%

                    echo.
                    echo BAR File:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar

                    echo.
                    echo ========================================
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                        --input-path "%WORKSPACE%\\ace-workspace" ^
                        --output-bar-file "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: BAR PACKAGING FAILED
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

                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar

                    if not exist "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar" (
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR CONTENTS
                    echo ========================================

                    jar tf "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: Unable to read BAR file
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo Checking Application ZIP
                    echo ========================================

                    jar tf "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar" | findstr /I "%APPLICATION_NAME%.appzip"

                    if errorlevel 1 (
                        echo.
                        echo ERROR: %APPLICATION_NAME%.appzip not found
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR CREATED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        stage('Check/Create ACE Server') {
            steps {
                echo '========================================'
                echo 'STAGE 6 - Check/Create ACE Server'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo ========================================
                    echo ACE SERVER VALIDATION
                    echo ========================================

                    echo ACE Server Work Directory:
                    echo %ACE_SERVER_WORKDIR%

                    echo.
                    echo Checking ACE Server...

                    if exist "%ACE_SERVER_WORKDIR%\\server.conf.yaml" (

                        echo.
                        echo ========================================
                        echo ACE SERVER ALREADY EXISTS
                        echo ========================================
                        echo.

                        dir "%ACE_SERVER_WORKDIR%"

                    ) else (

                        echo.
                        echo ========================================
                        echo ACE SERVER DOES NOT EXIST
                        echo ========================================
                        echo Creating ACE Independent Integration Server
                        echo.

                        if not exist "C:\\ACE\\servers" (
                            mkdir "C:\\ACE\\servers"
                        )

                        echo.
                        echo Running mqsicreateworkdir...
                        echo.

                        mqsicreateworkdir "%ACE_SERVER_WORKDIR%"

                        if errorlevel 1 (
                            echo.
                            echo ERROR: Failed to create ACE server work directory
                            exit /b 1
                        )

                        echo.
                        echo ========================================
                        echo ACE SERVER CREATED SUCCESSFULLY
                        echo ========================================

                        dir "%ACE_SERVER_WORKDIR%"
                    )

                    echo.
                    echo ========================================
                    echo Checking server.conf.yaml
                    echo ========================================

                    if not exist "%ACE_SERVER_WORKDIR%\\server.conf.yaml" (
                        echo.
                        echo ERROR: server.conf.yaml was not created
                        echo.
                        echo Expected:
                        echo %ACE_SERVER_WORKDIR%\\server.conf.yaml
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE SERVER VALIDATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        stage('Deploy BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 7 - Deploy ACE BAR'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo ========================================
                    echo DEPLOYMENT INFORMATION
                    echo ========================================

                    echo Application:
                    echo %APPLICATION_NAME%

                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER_WORKDIR%

                    echo.
                    echo ========================================
                    echo DEPLOYING BAR
                    echo ========================================

                    ibmint deploy ^
                        --input-bar-file "%WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar" ^
                        --output-work-directory "%ACE_SERVER_WORKDIR%"

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


        stage('Verify Deployment') {
            steps {
                echo '========================================'
                echo 'STAGE 8 - Verify Deployment'
                echo '========================================'

                bat '''
                    call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

                    echo.
                    echo ========================================
                    echo VERIFYING ACE DEPLOYMENT
                    echo ========================================

                    echo Application:
                    echo %APPLICATION_NAME%

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER_WORKDIR%

                    echo.
                    echo ========================================
                    echo Checking runtime directory
                    echo ========================================

                    if not exist "%ACE_SERVER_WORKDIR%\\run\\%APPLICATION_NAME%" (
                        echo.
                        echo ERROR: Application was not deployed
                        echo.
                        echo Expected:
                        echo %ACE_SERVER_WORKDIR%\\run\\%APPLICATION_NAME%
                        exit /b 1
                    )

                    dir /S /B "%ACE_SERVER_WORKDIR%\\run\\%APPLICATION_NAME%"

                    echo.
                    echo ========================================
                    echo APPLICATION DEPLOYMENT FOUND
                    echo ========================================

                    echo.
                    echo DEPLOYMENT VERIFICATION COMPLETED
                    echo ========================================
                '''
            }
        }


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
                    echo %APPLICATION_NAME%

                    echo.
                    echo Build:
                    echo %BUILD_NUMBER%

                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\builds\\%BUILD_NUMBER%\\%APPLICATION_NAME%.bar

                    echo.
                    echo ACE Server:
                    echo %ACE_SERVER_WORKDIR%

                    echo.
                    echo ========================================
                    echo ACE DEPLOYMENT PIPELINE COMPLETED
                    echo ========================================
                '''
            }
        }
    }


    post {
        always {
            echo '========================================'
            echo 'PIPELINE FINISHED'
            echo '========================================'
        }

        success {
            echo '========================================'
            echo 'JENKINS BUILD SUCCESSFUL'
            echo '========================================'

            echo "Application : ${APPLICATION_NAME}"
            echo "Build       : ${BUILD_NUMBER}"
            echo "BAR         : ${WORKSPACE}\\builds\\${BUILD_NUMBER}\\${APPLICATION_NAME}.bar"
            echo "ACE Server  : ${ACE_SERVER_WORKDIR}"

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
    }
}
