pipeline {
    agent any
    tools {
        jdk 'jdk-11'
        maven 'maven-3.9.12'
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
    }
    parameters {
        string(name: 'JSLEE_JAVAEE_VERSION', defaultValue: '7.1.14', description: 'The major version for JAIN SLEE JavaEE')
    }
    environment {
        JAVAEE_BUILD_VERSION = "${params.JSLEE_JAVAEE_VERSION}-${BUILD_NUMBER}"
    }
    stages {
        stage("Build") {
            steps {
                script {
                    JAVAEE_BUILD_VERSION = "${params.JSLEE_JAVAEE_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${JAVAEE_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE JavaEE (${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
            }
        }
        stage('Set Version') {
            steps {
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${JAVAEE_BUILD_VERSION}"
                sh "mvn versions:commit"
            }
        }
        stage("Release") {
            steps {
                sh "mvn clean install -Prelease -Drelease.dir=../../../${JAVAEE_BUILD_VERSION} -Dmaven.test.skip=true"
            }
        }
        stage('Zip Resources') {
            steps {
                dir("${JAVAEE_BUILD_VERSION}/examples") {
                    sh "zip -r slee-connectivity.zip slee-connectivity"
                    sh 'rm -rf slee-connectivity'
                }
            }
        }
        stage('Save Artifacts') {
            steps {
                archiveArtifacts artifacts: "${JAVAEE_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
        }
        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
            steps {
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.javaee/${JAVAEE_BUILD_VERSION}/"
                sh "cp -r ${JAVAEE_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.javaee/${JAVAEE_BUILD_VERSION}/"
                sh "rm -rf ${JAVAEE_BUILD_VERSION}"
            }
        }
    }
    post {
        success { echo "JAIN-SLEE JavaEE successfully built" }
        failure { echo "Building JAIN-SLEE JavaEE failed" }
    }
}
