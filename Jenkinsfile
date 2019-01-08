@Library('forge-shared-library')_

pipeline {
    options {
        disableConcurrentBuilds()
    }

    agent any

    environment {
        GRADLE_ARGS = '--no-daemon --console=plain'
    }

    stages {
        stage('Fetch') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew ${GRADLE_ARGS} --refresh-dependencies --continue :build'

                script {
                    env.MOD_VERSION = sh(returnStdout: true, script: './gradlew ${GRADLE_ARGS} :properties -q | grep "version:" | awk \'{print $2}\'').trim()
                }
            }

            post {
                success {
                    writeChangelog(currentBuild, 'build/changelog.txt')
                }
            }
        }

        stage('Publish') {
            when {
                not {
                    changeRequest()
                }
            }

            environment {
                MOD_MAVEN = credentials('mod-maven')
                MOD_KEYSTORE = credentials('mod-keystore')
                MOD_KEYSTORE_KEYPASS = credentials('mod-keystore-keypass')
                MOD_KEYSTORE_STOREPASS = credentials('mod-keystore-storepass')
                MOD_CURSE_FORGE = credentials('mod-curse-forge')
            }

            steps {
                script {
                    sh './gradlew ${GRADLE_ARGS} :publish -PmodMaven=${MOD_MAVEN_URL} -PmodMavenUser=${MOD_MAVEN_USR} -PmodMavenPassword=${MOD_MAVEN_PSW} -PmodKeystore=${MOD_KEYSTORE} -PmodKeystoreKeypass=${MOD_KEYSTORE_KEYPASS} -PmodKeystoreStorepass=${MOD_KEYSTORE_STOREPASS} -PmodCurseForge=${MOD_CURSE_FORGE}'
                }
            }
        }

        stage('Test PR') {
            when {
                changeRequest()
            }

            steps {
                script {
                    sh './gradlew ${GRADLE_ARGS} :publish
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/libs/**/*.*', fingerprint: true
        }
    }
}
