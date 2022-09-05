pipeline{
	agent { 
        node { 
            label 'nodo-java' 
        }
    }
	environment {
		DOCKERHUB_CREDENTIALS=credentials('docker-hub')
	}
	stages {
		stage('Build') {
			steps {
                sh 'mvn clean install'
				jacoco()
				nexusArtifactUploader artifacts: [[artifactId: 'spring-boot-jpa-h2', classifier: '', file: 'spring-boot-jpa-h2-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.bezkoder', nexusUrl: 'localhost:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'bootcamp', version: '0.0.1-SNAPSHOT'
				sh 'docker build -t dberenguerdevcenter/spring-boot-app:latest .'
			}
		}
		stage('Push Image to Docker Hub') {
			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
				sh 'docker push dberenguerdevcenter/spring-boot-app:latest'
			}
		}
		stage('Deploy to K8s')
		{
			steps{
				sh 'git clone https://github.com/dberenguerdevcenter/kubernetes-helm-docker-config.git configuracion --branch demo-java'
				sh 'kubectl apply -f configuracion/kubernetes-deployments/spring-boot-app/deployment.yaml --kubeconfig=configuracion/kubernetes-config/config'
			}
		}
	}
	post {
		always {
			sh 'docker logout'
		}
	}
}