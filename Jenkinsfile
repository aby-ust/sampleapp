def paths = "C:/ProgramData/jenkins/.jenkins/workspace/dotnetapps/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj"


pipeline 
{
    environment
    {
        appName = "aby-dotnetapp"
        resourceGroup = "Training-rg"
        
    }
    agent any	
	stages 
	{
        stage('sonar') 
        {
            steps
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'
                    withSonarQubeEnv('SonarQubeServer') 
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"Dotnetapp\""
                        
			                  bat "dotnet build ${paths}"    
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        /*stage("Quality gate") 
        {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'squser'
            }
        }*/
        stage('build')
        {
            steps
            {
                bat "dotnet build ${paths}"
		    
            }
        }
        stage('test')
        {
            steps
            {
                bat "dotnet test ${paths}"
		    
            }
        }
        stage('publish')
        {
            steps
            {
                bat "dotnet publish ${paths}"
		    
            }
        }
        
        stage('Package') 
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
                
                bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish"
                }
        }
        
        stage ('jfrog creation')
        {
            steps
            {
               rtServer (
                 id: "Artifactory",
                 
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }
        stage('Uploading file to jfrog')
        {
            steps{
                rtUpload (
                 serverId:"Artifactory" ,
                  spec: '''{
                   "files": [
                      {
                      "pattern": "*.zip",
                      "target": "Aby-dotnet-app"
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish build info') 
        {
            steps 
            {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        }
stage ('download the artifacts from artifactory')
        {
            steps
            {
                  rtDownload (
                    serverId: "Artifactory",
                        spec:
                              """{
                                "files": [
                                  {
                                    "pattern": "Aby-dotnet-app/*.zip",
                                    "target": "G:/jfrog/"          
                                  }
                               ]
                              }"""
      )
            }
        }
   /* stage('Extract ZIP') 
	{
	   steps
	   {
		   powershell '''
		                  Expand-Archive 'D:/jfrog/dotnetapp.zip' -DestinationPath 'D:/home/'
		              '''
        } 
	} */
	stage('Deploy to azure') 
	{
	   steps
		{
		   
			azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
	    }
	}
	}
}


	 