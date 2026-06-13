pipeline {

    agent any

    environment {
        ANDROID_HOME = "/opt/android-sdk"
        PATH = "${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/cmdline-tools/latest/bin:${env.PATH}"
        FIREBASE_APP_ID = "1:172568603780:android:51bc15e9950ba762e766d1"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Fix Permissions') {
            steps {
                sh 'chmod +x gradlew'
            }
        }

        stage('Add the local.properties file') {
            steps {
                sh 'echo "sdk.dir=/opt/android-sdk" > local.properties'
            }
        }        

        stage('Clean') {
            steps {
                sh './gradlew clean'
            }
        }


        stage('Build APK') {
            steps {
                sh './gradlew assembleRelease'
            }
        }

        stage('Archive APK') {
            steps {
                archiveArtifacts artifacts: 'app/build/outputs/apk/release/*.apk'
            }
        }


        stage('Install Firebase CLI') {
            steps {
                sh '''
                if ! command -v firebase >/dev/null 2>&1; then
                    echo "Firebase CLI not found, installing..."
                    curl -sL https://firebase.tools | bash
                else
                    echo "Firebase CLI already installed: $(firebase --version)"
                fi
                '''
            }
        }


        stage('Upload to Firebase App Distribution') {
            steps {
                withCredentials([file(credentialsId: 'firebase-service-account', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    APK_PATH=$(find app/build/outputs/apk/release -name "*.apk" | head -n 1)

                    firebase appdistribution:distribute "$APK_PATH" \
                      --app "$FIREBASE_APP_ID" \
                      --groups "testers" \
                      --release-notes "Automated Jenkins build #$BUILD_NUMBER"
                    '''
                }
            }
        }
    }
}