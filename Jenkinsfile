pipeline {
    agent none

    environment {
        MAJAOR_VERSION = 1
    }

    stages {
        stage('Unit Test') {
            agent {
                label 'apache'
            }
            steps {
                sh 'ant -f test.xml -v'
                junit 'reports/result.xml'
            }
        }
        stage('build') {
            agent {
                label 'apache'
            }
            steps {
                sh 'ant -f build.xml -v'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
                }
            }
        }
        stage('deploy') {
            agent {
                label 'apache'
            }
            steps {
                sh "if ![ -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
                sh "cp dist/rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
            }
        }
        stage("Running on CentOS") {
            agent {
                label 'CentOS'
            }
            steps {
                sh "wget http://snkanka4.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar"
                sh "java -jar rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
            }
        }
        stage("Test on Debian"){
            agent {
                docker 'openjdk:8u171-jre'
            }
            steps {
                sh "wget http://snkanka4.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar"
                sh "java -jar rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
            }
        }
        stage('Promote to green'){
            agent {
                label 'apache'
            }
            when {
                branch 'master'
            }
            steps {
                sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}.jar"
            }
        }
        stage('Promote Development branch to Master') {
            agent {
                label 'apache'
            }
            when {
                branch 'development'
            }
            steps {
                echo "Stashing Any Local Changes"
                sh 'git stash'
                echo "Checking Out Development Branch"
                sh 'git checkout development'
                echo 'Checking Out master Branch'
                sh 'git pull origin'
                echo 'Merging Development into Master Branch'
                sh 'git merge development'
                echo 'Pushing to Origin Master'
                sh 'git push origin master'
                echo 'Tagging the Release'
                sh "git tag rectangle-${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}"
                sh "git push origin rectangle-${env.MAJAOR_VERSION}.${env.BUILD_NUMBER}"
            }
        }
    }
}
