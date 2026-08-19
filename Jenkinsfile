pipeline {

    agent any

    environment {
        ACE_HOME = 'C:\\Program Files\\IBM\\ACE\\12.0.12.22'
        APP_NAME = 'GitHubPOCApp'
        BAR_DIR  = 'build'
        BAR_NAME = 'GitHubPOCApp.bar'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '========================================'
                echo 'STAGE 1 - GitHub Checkout'
                echo '========================================'

                checkout scm

                bat '''
                    echo.
                    echo Workspace:
                    echo %WORKSPACE%

                    echo.
                    echo Source files:
                    dir
                '''
            }
        }

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
                    echo Checking mqsicreatebar:
                    where mqsicreatebar

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
                    echo ACE project files validated successfully.
                '''
            }
        }

       stage('Create BAR') {
    steps {
        echo '========================================'
        echo 'STAGE 3 - Create BAR File'
        echo '========================================'

        bat '''
            echo.
            echo Creating ACE workspace...

            if exist "%WORKSPACE%\\ace-workspace" (
                rmdir /S /Q "%WORKSPACE%\\ace-workspace"
            )

            mkdir "%WORKSPACE%\\ace-workspace"
            mkdir "%WORKSPACE%\\ace-workspace\\%APP_NAME%"

            echo.
            echo Copying ACE project...

            xcopy "%WORKSPACE%\\.project" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y
            xcopy "%WORKSPACE%\\application.descriptor" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y
            xcopy "%WORKSPACE%\\*.msgflow" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y
            xcopy "%WORKSPACE%\\*.esql" "%WORKSPACE%\\ace-workspace\\%APP_NAME%\\" /Y

            echo.
            echo ACE workspace structure:

            dir /S /B "%WORKSPACE%\\ace-workspace"

            echo.
            echo Loading ACE environment...

            call "%ACE_HOME%\\server\\bin\\mqsiprofile.cmd"

            echo.
            echo Creating BAR:

            mqsicreatebar ^
                -data "%WORKSPACE%\\ace-workspace" ^
                -b "%WORKSPACE%\\%BAR_DIR%\\%BAR_NAME%" ^
                -p "%APP_NAME%" ^
                -cleanBuild

            if errorlevel 1 (
                echo.
                echo ERROR: BAR creation failed
                exit /b 1
            )

            echo.
            echo BAR creation completed successfully.
        '''
    }
}

        stage('Verify BAR') {
            steps {
                echo '========================================'
                echo 'STAGE 4 - Verify BAR'
                echo '========================================'

                bat '''
                    echo.
                    echo BAR directory:
                    dir "%WORKSPACE%\\%BAR_DIR%"

                    if not exist "%WORKSPACE%\\%BAR_DIR%\\%BAR_NAME%" (
                        echo.
                        echo ERROR: BAR file does not exist
                        exit /b 1
                    )

                    echo.
                    echo ========================================
                    echo BAR CREATED SUCCESSFULLY
                    echo ========================================
                    echo.
                    echo BAR:
                    echo %WORKSPACE%\\%BAR_DIR%\\%BAR_NAME%
                '''
            }
        }
    }

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
    }
}
