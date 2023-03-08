pipeline {
    agent { 
        node {
            label 'docker-tdp-builder'
            }
      }
    environment {
        number="${currentBuild.number}"
      }
    triggers {
        pollSCM '0 1 * * *'
      }
    stages {
        stage('clone') {
            steps {
                echo "Cloning..."
                git branch: 'v1.6.1-TDP-alliage', url: 'https://github.com/Yanis77240/knox'
            }
        }
        stage('Build') {
            steps {
                echo "Building..."
                sh '''
                mvn clean install -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae
                '''
            }
        }
        /*stage('Test') {
            steps {
                echo "Testing..."
                sh '''
                mvn clean test -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae --fail-never
                '''
            }
        }*/
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn deploy -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae -s settings.xml'
                }
            }        
        }
        stage("Publish tar.gz to Nexus") {
            steps {
                echo "Publish tar.gz..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh '''
                    curl -v -u $user:$pass --upload-file target/1.6.1-TDP-0.1.0-SNAPSHOT/knox-1.6.1-TDP-0.1.0-SNAPSHOT.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/knox/knox-1.6.1-TDP-0.1.0-SNAPSHOT-${number}.tar.gz
                    curl -v -u $user:$pass --upload-file target/1.6.1-TDP-0.1.0-SNAPSHOT/knoxshell-1.6.1-TDP-0.1.0-SNAPSHOT.tar.gz http://172.19.0.2:8081/repository/maven-tar-files/knox/knoxshell-1.6.1-TDP-0.1.0-SNAPSHOT-${number}.tar.gz
                    '''
                }
            }        
        }
    }
}