def filepath = "C:/ProgramData/jenkins/.jenkins/workspace/dotnetapps/aspnet-core-dotnet-core/aspnet-core-dotnet-core.csproj" //store source file path into a variable
def jfrogpath = "G:/jfrog/" //downloaded artifact fron jfrog
def packagepath = "aspnet-core-dotnet-core/bin/Debug/netcoreapp1.1/publish" //published file
//paths 
/*pipeline 
{
    environment                                            //specifies a sequence of key-value pairs which will be defined as environment variables for all steps, or stage-specific steps, 

    {
        appName = "aby-dotnetapp"
        resourceGroup = "Training-rg"
        
    }*/
    agent any	
	stages 
	{
        stage('sonar')                                //code analysis in sonarqube
        {
            steps
            {
                script
                {
                    
                    def scannerHome = tool 'SonarScanner for MSBuild'                 //define variable to store the installed tool.
                    withSonarQubeEnv('SonarQubeServer')                               //block that allows you to select the SonarQube server you want to interact with.
                    {
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" begin /k:\"Dotnetapp\""     //Connection details you have configured in Jenkins global configuration will be automatically passed to the scanner.
                        
			                  bat "dotnet build ${filepath}"    
                        bat "\"${scannerHome}\\SonarScanner.MSBuild.exe\" end"
                    }
                }
            }
        }
        stage("Quality gate")         //check the quality gate status
        {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        stage('build')                             // Builds the project and its dependencies into a set of binaries.dotnet build uses MSBuild to build the project, so it supports both parallel and incremental builds
        {
            steps
            {
                bat "dotnet build ${filepath}"
		    
            }
        }
        stage('test')                      //The dotnet test command is used to execute unit tests in a given solution. The dotnet test command builds the solution and runs a test host application for each test project in the solution. 
        {
            steps
            {
                bat "dotnet test ${filepath}"
		    
            }
        }
        stage('publish')                  //The dotnet publish command's output is ready for deployment to a hosting system.dotnet publish compiles the application, reads through its dependencies specified in the project file, and publishes the resulting set of files to a directory.
        {
            steps
            {
                bat "dotnet publish ${filepath}"
		    
            }
        }
        
        stage('Package')                      //package the published files.
        {
            steps 
                {
                echo "Deploying to stage environment for more tests!";
                bat "del *.zip"
                
                bat "tar.exe -a -c -f WebApp_${BUILD_NUMBER}.zip ${packagepath}"       //packaged Zip file name with buildnumber.${BUILD_NUMBER} is a predefined function in jenkind.it invoke the current build number.
                }
        }
        
        /*stage ('jfrog creation')
        {
            steps
            {
               rtServer (
                 id: "Artifactory",
                 
                  bypassProxy: true,
                   timeout: 300
                        )
            }
        }*/
        stage('Uploading file to jfrog')     //This allows managing the File Specs in a source control, possible with the project sources.
        {
            steps{
                rtUpload (
                 serverId:"Artifactory" ,               //Creating an Artifactory Server Instance, server id is configured in manage jenkins setting
                  spec: '''{
                   "files": [
                      {
                      "pattern": "${WORKSPACE}/WebApp_${BUILD_NUMBER}.zip",
                      "target": "Aby-dotnet-app"                       //repo created in jfrog
                      }
                            ]
                           }''',
                        )
            }
        }
        stage ('Publish build info')     //provide url of published build information
        {
            steps 
            {
                rtPublishBuildInfo (
                    serverId: "Artifactory"
                )
            }
        }
stage ('download the artifacts from artifactory')     //File Spec which specifies the files which files should be downloaded & target path.
        {
            steps
            {
                  rtDownload (
                    serverId: "Artifactory",
                        spec:
                              """{
                                "files": [
                                  {
                                    "pattern": "Aby-dotnet-app/WebApp_${BUILD_NUMBER}.zip",
                                    "target": "${jfrogpath}"          
                                  }
                               ]
                              }"""
      )
            }
        }
   /* stage('Extract ZIP')  //this stage for deploy the application into iis server
	{
	   steps
	   {
		   powershell '''
		                  Expand-Archive 'D:/jfrog/dotnetapp.zip' -DestinationPath 'D:/home/'
		              '''
        } 
	} */
	stage('Deploy to azure')           //Depoly the published files into the azure webapp
	{
	   steps
		{
		   
			//azureWebAppPublish appName: "${env.appName}", azureCredentialsId: 'Azure', resourceGroup: "${env.resourceGroup}"
			  azureWebAppPublish azureCredentialsId: params.Credentials_id , resourceGroup: params.ResourceGroup , appName: params.Myapp
	    }
	}
	}
}


	 
