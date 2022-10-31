pipeline{

    agent {
        node {
            label "nodo-java"
        }
    }

    environment {
        registryCredential='docker-hub-credentials'
        registryBackend = 'franaznarteralco/backend-demo'
    }

    stages {

//         stage('SonarQube analysis') {
//           steps {
//             withSonarQubeEnv(credentialsId: "sonarqube-credentials", installationName: "sonarqube-server"){
//                 sh "mvn clean verify sonar:sonar -DskipTests"
//             }
//           }
//         }
//
//         stage('Quality Gate') {
//             steps {
//                 timeout(time: 10, unit: "MINUTES") {
//                     script {
//                         def qg = waitForQualityGate(webhookSecretId: 'sonarqube-credentials')
//                         if (qg.status != 'OK') {
//                             error "Pipeline aborted due to quality gate failure: ${qg.status}"
//                         }
//                     }
//                 }
//             }
//         }
//
//         stage('SonarQube analysis') {
//             steps {
//                 sh "mvn clean install -DskipTests"
//             }
//         }
//
//         stage('Push Image to Docker Hub') {
//             steps {
//                 script {
//                     dockerImage = docker.build registryBackend + ":$BUILD_NUMBER"
//                     docker.withRegistry( '', registryCredential) {
//                         dockerImage.push()
//                     }
//                 }
//             }
//         }
//
//         stage('Push Image latest to Docker Hub') {
//             steps {
//                 script {
//                     dockerImage = docker.build registryBackend + ":latest"
//                     docker.withRegistry( '', registryCredential) {
//                         dockerImage.push()
//                     }
//                 }
//             }
//         }
//
//         stage("Deploy to K8s"){
//             steps{
//                 script {
//                     if(fileExists("configuracion")){
//                         sh 'rm -r configuracion'
//                     }
//                 }
//                 sh 'git clone https://github.com/dberenguerdevcenter/kubernetes-helm-docker-config.git configuracion --branch test-implementation'
//                 sh 'kubectl apply -f configuracion/kubernetes-deployment/spring-boot-app/manifest.yml -n default --kubeconfig=configuracion/kubernetes-config/config'
//             }
//         }
//
//         stage ("Run API Test") {
//             steps{
//                 node("node-nodejs"){
//                     script {
//                         if(fileExists("spring-boot-app")){
//                             sh 'rm -r spring-boot-app'
//                         }
//                         sleep 15 // seconds
//                         sh 'git clone https://github.com/dberenguerdevcenter/spring-boot-app.git spring-boot-app --branch api-test-implementation'
//                         sh 'newman run spring-boot-app/src/main/resources/bootcamp.postman_collection.json --reporters cli,junit --reporter-junit-export "newman/report.xml"'
//                         junit "newman/report.xml"
//                     }
//                 }
//             }
//         }

        stage ("Run Performance Test") {
            steps{
                script {

                    if(fileExists("jmeter-docker")){
                       sh 'rm -r jmeter-docker'
                    }

                    sh 'git clone https://github.com/FranAznarTeralco/jmeter-docker.git'

                     dir('jmeter-docker') {

                        if(fileExists("apache-jmeter-5.5.tgz")){
                            sh 'rm -r apache-jmeter-5.5.tgz'
                        }

                        sh 'wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.5.tgz'
                        sh 'tar xvf apache-jmeter-5.5.tgz'
                        sh 'cp plugins/*.jar apache-jmeter-5.5/lib/ext'
                        sh 'mkdir test'
                        sh 'mkdir apache-jmeter-5.5/test'
                        sh 'cp ../src/main/resources/*.jmx apache-jmeter-5.5/test/'
                        sh 'chmod +775 ./build.sh && chmod +775 ./run.sh && chmod +775 ./entrypoint.sh'
                        sh 'rm -r apache-jmeter-5.5.tgz'
                        sh 'tar -czvf apache-jmeter-5.5.tgz apache-jmeter-5.5'
                        sh './build.sh'
                        sh 'rm -r apache-jmeter-5.5 && rm -r apache-jmeter-5.5.tgz'
                        sh 'cp ../src/main/resources/perform_test.jmx test'
                        sh './run.sh -n -t test/perform_test.jmx -l test/perform_test.jtl -Jthreads=2 -Jrampup=1 -Jduration=10'
                        sh 'docker cp jmeter:/home/jmeter/apache-jmeter-5.5/test/perform_test.jtl /home/jenkins/workspace/_app_perform-test-implementation/jmeter-docker/test'
                        perfReport '/home/jenkins/workspace/_app_perform-test-implementation/jmeter-docker/test/perform_test.jtl'
                     }

                }
            }
        }


        stage ("Generate Taurus Report") {
            steps{
                script {

                     dir('jmeter-docker') {
                        sh 'pip install bzt'
                            BlazeMeterTest: {
                                sh 'bzt  test/perform_test.jtl -report'
                            }
                     }
                }
            }
        }
    }

	post {
        always {
            sh "docker logout"
        }
	}

}