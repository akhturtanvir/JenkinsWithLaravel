pipeline {
    agent any

    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
    }

    environment {
        // CHANGE this: Your local Laragon PHP binary
        PHP_PATH = "C:\\laragon\\bin\\php\\php-8.2.6-Win32-vs16-x64\\php.exe"

        // CHANGE this: Composer path (if installed via ComposerSetup.exe)
        COMPOSER_PATH = "C:\\ProgramData\\ComposerSetup\\bin\\composer.bat"

        // CHANGE this: Laragon deployment folder
        DEPLOY_PATH = "C:\\laragon\\www\\JenkinsWithLaravel"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],  // ❗ Change branch if needed
                    userRemoteConfigs: [[
                        url: 'https://github.com/akhturtanvir/JenkinsWithLaravel.git',  // ❗ Change repo URL
                        credentialsId: 'github-token'  // ❗ Add this credential in Jenkins
                    ]]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                bat """
                    %COMPOSER_PATH% install --no-interaction --prefer-dist
                """
            }
        }

        stage('Laravel Optimization') {
            steps {
                bat """
                    %PHP_PATH% artisan config:cache
                    %PHP_PATH% artisan route:cache
                    %PHP_PATH% artisan view:cache
                """
            }
        }

        // stage('Run Tests (Optional)') {
        //     when {
        //         expression { fileExists('tests') }
        //     }
        //     steps {
        //         bat """
        //             %PHP_PATH% vendor\\bin\\phpunit
        //         """
        //     }
        // }

        stage('Deploy to Laragon') {
            steps {
                bat """
                    echo Deploying to Laragon...
                    robocopy . "%DEPLOY_PATH%" /MIR /XF .env /XD vendor node_modules .git
                """
            }
        }

    }

    post {
        success {
            echo "Laravel App Deployed Successfully!"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
