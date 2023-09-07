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
            stage('Git Clone') {
                echo "Cloning.."
                git branch: 'v1.6.1-TDP-alliage-k8s', url: 'https://github.com/Yanis77240/knox'
            }
            stage('Chose comparison') {
                withEnv(["file=${input message: 'Select file in http://10.10.10.11:30000/repository/component-test-comparison/', parameters: [string('number of results file')]}"]) {
                    withEnv(["number=${currentBuild.number}"]) {
                        sh '''
                        cd test-comparison
                        curl -v http://10.110.4.212:8081/repository/component-test-comparison/knox-1.6/${file} > ${file}
                        python3 src/python/comparison_file_check.py ${file}
                        echo "python3 src/python/main.py 2.20 ${number} ${file}" > transformation.sh
                        chmod 777 transformation.sh
                        '''
                    }
                }
            }
            stage ('Build') {
                echo "Building.."
                sh '''
                mvn clean install -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae -Dstyle.color=never
                '''
            }
            stage('Test') {
                echo "Testing..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    withEnv(["number=${currentBuild.number}"]) {
                        /* Perform the tests and the surefire reporting*/
                        sh '''
                        mvn clean install --batch-mode  --fail-never -Dstyle.color=never | tee output.txt
                        '''
                        sh 'mvn surefire-report:report-only  -Daggregate=true'
                        sh 'curl -v -u $user:$pass --upload-file target/site/surefire-report.html http://10.110.4.212:8081/repository/test-reports/knox/surefire-report-${number}.html'
                        /* extract the java-test and scalatest-plugin data output and remove all color signs */
                        sh'./test-comparison/src/grep-commands/grep-surefire-2.20.sh'
                        /*sh'./test-comparison/src/grep-commands/grep-scalatest.sh'*/
                        /* Perform the data transformation and the comparison*/
                        sh '''
                        cd test-comparison
                        ./transformation.sh
                        ./src/decision.sh ${number}
                        curl -v -u $user:$pass --upload-file results-${number}.json http://10.110.4.212:8081/repository/component-test-comparison/knox-1.6/results-${number}.json
                        '''
                    }
                }
            }
            stage('Deliver') {
                echo "Deploy..."
                withCredentials([usernamePassword(credentialsId: '4b87bd68-ad4c-11ed-afa1-0242ac120002', passwordVariable: 'pass', usernameVariable: 'user')]) {
                    sh 'mvn clean deploy -Prelease -Ppackage -Drat.numUnapprovedLicenses=1000 -DskipTests -Dmaven.javdoc.skip=true -Dcheckstyle.skip=true -Dfindbugs.skip=true -Dspotbugs.skip=true --batch-mode -fae -Dstyle.color=never -s settings.xml'
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