pipeline{
	agent any
	environment {
		DOCKERHUB_CREDENTIALS=credentials('docker-hub')
	}
	stages {
		stage('Build') {
			steps {
                sh 'mvn clean install -f pom.xml'
				sh 'docker build -t dberenguerdevcenter/spring-boot-app:latest .'
			}
		}
		stage('Login Docker Hub') {
			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}
		stage('Push Image to Docker Hub') {
			steps {
				sh 'docker push dberenguerdevcenter/spring-boot-app:latest'
			}
		}
		stage('Deploy to K8s')
		{
			steps{
				sh 'git clone https://github.com/dberenguerdevcenter/kubernetes-helm-docker-config.git configuracion --branch master'
				sh 'kubectl apply -f configuracion/kubernetes-deployments/spring-boot-app/deployment.yaml --kubeconfig=configuracion/kubernetes-config/cluster-david/config'
			}
		}
	}
	post {
		always {
			sh 'docker logout'
		}
	}
}