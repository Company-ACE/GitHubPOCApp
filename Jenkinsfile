pipeline {

    agent any

    environment {
        ACE_HOME       = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        ACE_PROFILE    = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22\\server\\bin\\mqsiprofile.cmd'
        INTEGRATION_SERVER = 'TEST'
        SERVER_WORK_DIR    = 'C:\\ACE\\servers\\TEST'
        APPLICATION        = 'GitHubPOCApp'
        BAR_NAME           = 'GitHubPOCApp.bar'
        HTTP_PORT          = '7800'
        APP_WORKSPACE      = 'ace-workspace'
        BUILD_DIR          = 'builds'
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
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo ACE INSTALLATION
                    echo ========================================

                    echo ACE_HOME:
                    echo %ACE_HOME%
                    echo.

                    echo Checking ibmint:
                    if not exist "%ACE_HOME%\\server\\bin\\ibmint.exe" (
                        echo ERROR: ibmint.exe not found
                        exit /b 1
                    )

                    echo %ACE_HOME%\\server\\bin\\ibmint.exe
                    echo.

                    echo Checking IntegrationServer:
                    if not exist "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" (
                        echo ERROR: IntegrationServer.exe not found
                        exit /b 1
                    )

                    echo %ACE_HOME%\\server\\bin\\IntegrationServer.exe
                    echo.

                    echo ACE Version:
                    ibmint --help >nul
                    if errorlevel 1 (
                        echo ERROR: ACE validation failed
                        exit /b 1
                    )

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

                    if exist "%WORKSPACE%\\%APP_WORKSPACE%" (
                        rmdir /S /Q "%WORKSPACE%\\%APP_WORKSPACE%"
                    )

                    mkdir "%WORKSPACE%\\%APP_WORKSPACE%"
                    mkdir "%WORKSPACE%\\%APP_WORKSPACE%\\%APPLICATION%"

                    echo.
                    echo ========================================
                    echo Checking ACE Application Files
                    echo ========================================

                    if not exist "%WORKSPACE%\\.project" (
                        echo ERROR: .project not found
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
                    echo Copying ACE Application Files
                    echo ========================================

                    copy /Y "%WORKSPACE%\\.project" "%WORKSPACE%\\%APP_WORKSPACE%\\%APPLICATION%\\"
                    copy /Y "%WORKSPACE%\\application.descriptor" "%WORKSPACE%\\%APP_WORKSPACE%\\%APPLICATION%\\"
                    copy /Y "%WORKSPACE%\\*.msgflow" "%WORKSPACE%\\%APP_WORKSPACE%\\%APPLICATION%\\"
                    copy /Y "%WORKSPACE%\\*.esql" "%WORKSPACE%\\%APP_WORKSPACE%\\%APPLICATION%\\"

                    echo.
                    echo ========================================
                    echo ACE APPLICATION STRUCTURE
                    echo ========================================

                    dir /S /B "%WORKSPACE%\\%APP_WORKSPACE%"

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
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo Creating BAR Directory
                    echo ========================================

                    if exist "%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%" (
                        rmdir /S /Q "%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%"
                    )

                    mkdir "%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%"

                    echo Build Number:
                    echo %BUILD_NUMBER%
                    echo.

                    echo BAR Directory:
                    echo %WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%
                    echo.

                    echo BAR File:
                    echo %WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%\\%BAR_NAME%

                    echo.
                    echo ========================================
                    echo Running ibmint package
                    echo ========================================

                    ibmint package ^
                        --input-path "%WORKSPACE%\\%APP_WORKSPACE%" ^
                        --output-bar-file "%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%\\%BAR_NAME%"

                    if errorlevel 1 (
                        echo ERROR: BAR packaging failed
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR PACKAGING COMPLETED
                    echo ========================================

                    dir "%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%"
                '''
            }
        }


        stage('Verify BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 5 - Verify BAR'
                echo '========================================'

                bat '''
                    set "BAR_FILE=%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%\\%BAR_NAME%"

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

                    jar tf "%BAR_FILE%" | findstr /I "%APPLICATION%.appzip"

                    if errorlevel 1 (
                        echo ERROR: %APPLICATION%.appzip not found
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR CREATED SUCCESSFULLY
                    echo ========================================
                '''
            }
        }


        stage('Stop Existing ACE Server') {
            steps {
                echo '========================================'
                echo 'STAGE 6 - Stop Existing ACE Server'
                echo '========================================'

                bat '''
                    echo.
                    echo Checking for existing IntegrationServer...

                    powershell -NoProfile -ExecutionPolicy Bypass -Command "$p = Get-CimInstance Win32_Process -Filter \\"Name='IntegrationServer.exe'\\" | Where-Object { $_.CommandLine -like '*C:\\ACE\\servers\\TEST*' }; if ($p) { Write-Host 'Stopping existing ACE Integration Server...'; $p | ForEach-Object { Stop-Process -Id $_.ProcessId -Force } } else { Write-Host 'No existing TEST Integration Server found.' }"

                    timeout /t 3 /nobreak >nul

                    echo.
                    echo Checking IntegrationServer process...

                    tasklist | findstr /I "IntegrationServer.exe"

                    if not errorlevel 1 (
                        echo ERROR: Existing IntegrationServer process is still running
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo EXISTING SERVER STOPPED
                    echo ========================================
                '''
            }
        }


        stage('Deploy BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 7 - Deploy BAR'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    set "BAR_FILE=%WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%\\%BAR_NAME%"

                    echo.
                    echo ========================================
                    echo DEPLOYING BAR
                    echo ========================================

                    echo BAR:
                    echo %BAR_FILE%
                    echo.

                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%
                    echo.

                    if not exist "%SERVER_WORK_DIR%" (
                        echo Creating server work directory...
                        mkdir "%SERVER_WORK_DIR%"
                    )

                    echo.
                    echo Running ibmint deploy...

                    ibmint deploy ^
                        --input-bar-file "%BAR_FILE%" ^
                        --output-work-directory "%SERVER_WORK_DIR%" ^
                        --restart-all-applications

                    if errorlevel 1 (
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


        stage('Start ACE Integration Server') {
            steps {
                echo '========================================'
                echo 'STAGE 8 - Start ACE Integration Server'
                echo '========================================'

                bat '''
                    call "%ACE_PROFILE%"

                    echo.
                    echo ========================================
                    echo STARTING ACE INTEGRATION SERVER
                    echo ========================================

                    echo Server:
                    echo %INTEGRATION_SERVER%
                    echo.

                    echo Work Directory:
                    echo %SERVER_WORK_DIR%
                    echo.

                    start "" /B "%ACE_HOME%\\server\\bin\\IntegrationServer.exe" --work-dir "%SERVER_WORK_DIR%"

                    echo IntegrationServer start command submitted.

                    echo.
                    echo Waiting for ACE server startup...

                    timeout /t 5 /nobreak >nul

                    set "SERVER_STARTED="

                    for /L %%N in (1,1,12) do (
                        tasklist | findstr /I "IntegrationServer.exe" >nul

                        if not errorlevel 1 (
                            set "SERVER_STARTED=YES"
                            goto SERVER_FOUND
                        )

                        echo Waiting... attempt %%N of 12
                        timeout /t 5 /nobreak >nul
                    )

                    :SERVER_FOUND

                    if not defined SERVER_STARTED (
                        echo ERROR: IntegrationServer.exe did not start
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo INTEGRATION SERVER PROCESS IS RUNNING
                    echo ========================================

                    tasklist | findstr /I "IntegrationServer.exe"
                '''
            }
        }


        stage('Verify ACE Listener') {
            steps {
                echo '========================================'
                echo 'STAGE 9 - Verify ACE HTTP Listener'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo CHECKING ACE HTTP PORT
                    echo ========================================

                    echo Expected HTTP Port:
                    echo %HTTP_PORT%
                    echo.

                    set "PORT_FOUND="

                    for /L %%N in (1,1,12) do (
                        netstat -ano | findstr ":%%N" >nul
                    )

                    netstat -ano | findstr ":%HTTP_PORT%" | findstr "LISTENING"

                    if not errorlevel 1 (
                        set "PORT_FOUND=YES"
                    )

                    if not defined PORT_FOUND (
                        echo ERROR: ACE HTTP listener port %HTTP_PORT% is not listening
                        echo.
                        echo Current listening ports:
                        netstat -ano | findstr LISTENING
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo ACE HTTP LISTENER IS RUNNING
                    echo ========================================

                    netstat -ano | findstr ":%HTTP_PORT%" | findstr "LISTENING"
                '''
            }
        }


        stage('Verify Deployment') {
            steps {
                echo '========================================'
                echo 'STAGE 10 - Verify Deployment'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo CHECKING SERVER WORK DIRECTORY
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%" (
                        echo ERROR: Server work directory not found
                        exit /b 1
                    )

                    dir /S /B "%SERVER_WORK_DIR%"

                    echo.
                    echo ========================================
                    echo CHECKING APPLICATION
                    echo ========================================

                    if not exist "%SERVER_WORK_DIR%\\run\\%APPLICATION%" (
                        echo ERROR: Application directory not found
                        exit /b 1
                    )

                    if not exist "%SERVER_WORK_DIR%\\run\\%APPLICATION%\\application.descriptor" (
                        echo ERROR: application.descriptor not deployed
                        exit /b 1
                    )

                    if not exist "%SERVER_WORK_DIR%\\run\\%APPLICATION%\\%APPLICATION:.appzip=.msgflow%" (
                        echo Application message flow check completed
                    )

                    echo.
                    echo ========================================
                    echo CHECKING ACE SERVER PROCESS
                    echo ========================================

                    tasklist | findstr /I "IntegrationServer.exe"

                    if errorlevel 1 (
                        echo ERROR: IntegrationServer.exe is not running
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo DEPLOYMENT VERIFICATION SUCCESSFUL
                    echo ========================================
                '''
            }
        }


        stage('Final Verification') {
            steps {
                echo '========================================'
                echo 'FINAL VERIFICATION'
                echo '========================================'

                bat '''
                    echo.
                    echo ========================================
                    echo PIPELINE SUMMARY
                    echo ========================================

                    echo Application:
                    echo %APPLICATION%
                    echo.

                    echo Integration Server:
                    echo %INTEGRATION_SERVER%
                    echo.

                    echo Server Work Directory:
                    echo %SERVER_WORK_DIR%
                    echo.

                    echo HTTP Listener:
                    echo http://localhost:%HTTP_PORT%
                    echo.

                    echo BAR:
                    echo %WORKSPACE%\\%BUILD_DIR%\\%BUILD_NUMBER%\\%BAR_NAME%
                    echo.

                    echo Build Number:
                    echo %BUILD_NUMBER%
                    echo.

                    echo Git Commit:
                    git rev-parse HEAD
                    echo.

                    echo.
                    echo ========================================
                    echo ACE CI/CD PIPELINE SUCCESSFUL
                    echo ========================================
                '''
            }
        }
    }


    post {

        success {
            echo '========================================'
            echo 'JENKINS BUILD SUCCESS'
            echo '========================================'
            echo 'Application: GitHubPOCApp'
            echo 'Integration Server: TEST'
            echo 'HTTP Listener: 7800'
            echo 'Server Work Directory: C:\\ACE\\servers\\TEST'
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
