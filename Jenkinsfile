podTemplate(containers: [
    containerTemplate(
        name: 'tdp-builder', 
        image: 'yanisbariteau/tdp-builder:jenkins', 
        command: 'sleep', 
        args: '30d'
        )
  ]) {

    node(POD_LABEL) {
        container('tdp-builder') {
            environment {
                number="${currentBuild.number}"
            }
            stage('Git Clone') {
                echo "Cloning.."
                git branch: 'v1.6.1-TDP-alliage-k8s', url: 'https://github.com/Yanis77240/knox'
            }   
            stage ('Build') {
                echo "Building.."
                sh '''
                mvn clean install -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae
                '''
            }
            stage('Test') {
                echo "Testing.."
                sh '''
                mvn clean install --batch-mode
                '''
            }
            stage('Deliver') {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean install --batch-mode -Dsurefire.rerunFailingTestsCount=3'
                }
            }
            stage("Publish tar.gz to Nexus") {
                echo "Publish tar.gz..."
                withEnv(["number=${currentBuild.number}"]) {
                    withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh '''
                        curl -v -u $user:$pass --upload-file target/1.6.1-TDP-0.1.0-SNAPSHOT/knox-1.6.1-TDP-0.1.0-SNAPSHOT.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/knox/knox-1.6.1-TDP-0.1.0-SNAPSHOT-${number}.tar.gz
                        curl -v -u $user:$pass --upload-file target/1.6.1-TDP-0.1.0-SNAPSHOT/knoxshell-1.6.1-TDP-0.1.0-SNAPSHOT.tar.gz http://10.110.4.212:8081/repository/maven-tar-files/knox/knoxshell-1.6.1-TDP-0.1.0-SNAPSHOT-${number}.tar.gz
                        '''
                    }
                }
            }       
        }
    }
}