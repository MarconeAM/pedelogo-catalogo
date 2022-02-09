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
			    withSonarQubeEnv('sonarqube') {
				                        //  bat 'dotnet tool install --global dotnet-sonarscanner --version 5.5.1'
				                        //  bat 'dotnet tool install --global dotnet-sonarscanner'
					                  bat 'dotnet  SonarScanner.MSBuild.dll begin /k:"projetojpedelogo-pipeline" /d:sonar.host.url="http://localhost:9000"  /d:sonar.login="871535c71e2ae3e4f066c020911f9c1b71a944fa"'
				                          bat 'dotnet build PedeLogo.Catalogo.sln'
                                                          bat 'dotnet SonarScanner.MSBuild.dll end /d:sonar.login="871535c71e2ae3e4f066c020911f9c1b71a944fa"'
				                        
				     //   dotnet <path to SonarScanner.MSBuild.dll> begin /k:"project-key" /d:sonar.login="<token>"
					//dotnet build <path to solution.sln>
					//dotnet <path to SonarScanner.MSBuild.dll> end /d:sonar.login="<token>" 
       			    
       			   
                          	
                 }
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
