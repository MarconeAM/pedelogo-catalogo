pipeline {
    agent any

    stages {

        stage('Get Source') {
            steps {
                git url: 'https://github.com/MarconeAM/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Docker Build Image') {
            steps {
                script {
                    dockerapp = docker.build("marcone29/api-pedelogocatalogo:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }
       
        
        stage('SonarQube analysis') {
                    steps {
			     script {
			          withSonarQubeEnv('sonarqube') {
				                                dotnet sonarscanner begin /k:"projetojpedelogo-pipeline" /d:sonar.host.url="http://localhost:9000"  /d:sonar.login="871535c71e2ae3e4f066c020911f9c1b71a944fa"
				                                dotnet build PedeLogo.Catalogo.sln
                                                                dotnet sonarscanner end /d:sonar.login="871535c71e2ae3e4f066c020911f9c1b71a944fa"
                       	   }
   		    }
	       }
          }

		stage('Email Sucess')
		{
                         steps {
			emailext (
				to: 'mrcn.alvesmiranda@gmail.com',
				subject: "Sucess Pipeline: ${currentBuild.fullDisplayName}",
				body: "Sucess with ${env.BUILD_URL}"
				)
			 }

        }
        
        stage('Email Failed')
		{
                   steps {
			emailext (
				to: 'mrcn.alvesmiranda@gmail.com, marcone.alves@fsfx.com.br',
				subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
				body: "Something is wrong with ${env.BUILD_URL}"
				)	 
		   }
		}
        
        stage('Deploy Kubernetes') {
            agent {
                kubernetes {
                    cloud 'kubernetes'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }

            steps {
                sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api.yaml'
                sh 'cat ./k8s/api.yaml'
                kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
            }
        }
    }
}
