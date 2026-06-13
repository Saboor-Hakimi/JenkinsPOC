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
                    APK_PATH=$(find app/build/outputs/apk/release -name "*.apk" | head -n 1)

                    TAG="build-${BUILD_NUMBER}"

                    echo "Creating release $TAG"

                    curl -X POST \
                    -H "Authorization: Bearer $GITHUB_TOKEN" \
                    -H "Accept: application/vnd.github+json" \
                    https://api.github.com/repos/Saboor-Hakimi/JenkinsPOC/releases \
                    -d "{
                        \"tag_name\": \"$TAG\",
                        \"name\": \"$TAG\",
                        \"body\": \"Automated build from Jenkins #$BUILD_NUMBER\"
                    }"
                    '''
                }
            }
        }


       stage('Upload APK to Release') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {

                    sh '''
                    APK_PATH=$(find app/build/outputs/apk/release -name "*.apk" | head -n 1)

                    TAG="build-${BUILD_NUMBER}"

                    # Get upload URL
                    UPLOAD_URL=$(curl -s \
                    -H "Authorization: Bearer $GITHUB_TOKEN" \
                    https://api.github.com/repos/Saboor-Hakimi/JenkinsPOC/releases/tags/$TAG \
                    | grep upload_url | cut -d '"' -f 4 | cut -d '{' -f 1)

                    echo "Uploading APK..."

                    curl -X POST \
                    -H "Authorization: Bearer $GITHUB_TOKEN" \
                    -H "Content-Type: application/vnd.android.package-archive" \
                    --data-binary @"$APK_PATH" \
                    "$UPLOAD_URL?name=app-release.apk"
                    '''
                }
            }
        } 
    }
}