pipeline {
    agent any

    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
    }

    environment {
        // PHP executable from Laragon
        PHP_PATH = "C:\\laragon\\bin\\php\\php-8.2.6-Win32-vs16-x64\\php.exe"

        // Composer installed via ComposerSetup.exe
        COMPOSER_PATH = "C:\\ProgramData\\ComposerSetup\\bin\\composer.bat"

        // Laragon project directory (deployment location)
        DEPLOY_PATH = "C:\\laragon\\www\\JenkinsWithLaravel"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                echo "📥 Checking out source code..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],   // Change branch if needed
                    userRemoteConfigs: [[
                        url: 'https://github.com/akhturtanvir/JenkinsWithLaravel.git',
                        credentialsId: 'github-token'
                    ]]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "📦 Installing Composer dependencies..."
                bat """
                    %COMPOSER_PATH% install --no-interaction --prefer-dist
                """
            }
        }

        stage('Laravel Optimization') {
            steps {
                echo "⚡ Running Laravel optimization commands..."
                bat """
                    %PHP_PATH% artisan config:clear
                    %PHP_PATH% artisan cache:clear
                    %PHP_PATH% artisan route:clear
                    %PHP_PATH% artisan view:clear

                    %PHP_PATH% artisan config:cache
                    %PHP_PATH% artisan route:cache
                    %PHP_PATH% artisan view:cache
                """
            }
        }

        stage('Deploy to Laragon') {
            steps {
                echo "🚀 Deploying updated project to Laragon..."

                // Use robocopy (Windows) to sync workspace → Laragon
                bat """
                    robocopy . "%DEPLOY_PATH%" /MIR ^
                    /XF .env ^
                    /XD vendor node_modules .git
                """

                echo "🔁 Rebuilding vendor folder for deployed app..."
                bat """
                    cd %DEPLOY_PATH%
                    %COMPOSER_PATH% install --no-interaction --prefer-dist
                """

                echo "🔧 Re-optimizing deployed application..."
                bat """
                    cd %DEPLOY_PATH%
                    %PHP_PATH% artisan optimize
                """
            }
        }
    }

    post {
        success {
            echo "🎉 SUCCESS: Laravel Application Deployed Successfully to Laragon!"
        }
        failure {
            echo "❌ FAILURE: Something went wrong during deployment!"
        }
    }
}
