pipeline {

    agent any

    environment {
        ANDROID_HOME = "/opt/android-sdk"
        PATH = "${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/cmdline-tools/latest/bin:${env.PATH}"
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


        stage('Create GitHub Release') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {

                    sh '''
                    echo $GITHUB_TOKEN | gh auth login --with-token

                    TAG="build-${BUILD_NUMBER}"

                    gh release create "$TAG" \
                    --title "$TAG" \
                    --notes "Automated Jenkins build #$BUILD_NUMBER"
                    '''
                }
            }
        }


       stage('Upload APK') {
            steps {
                sh '''
                APK_PATH=$(find app/build/outputs/apk/release -name "*.apk" | head -n 1)
                TAG="build-${BUILD_NUMBER}"

                gh release upload "$TAG" "$APK_PATH" --clobber
                '''
            }
        } 
    }
}