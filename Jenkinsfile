pipeline {
    agent { 
        node {
            label 'docker-tdp-builder'
            }
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
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae -s settings.xml'
                }
            }        
        }
    }
}