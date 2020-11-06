def readProperties()
{
        def properties_file_path = "${workspace}" + "@script/properties.yml"
        def property = readYaml file: properties_file_path
        env.APP_NAME = property.APP_NAME
        env.MS_NAME = property.MS_NAME
        env.BRANCH = property.BRANCH
        env.GIT_SOURCE_URL = property.GIT_SOURCE_URL
        env.SONAR_HOST_URL = property.SONAR_HOST_URL
        env.CODE_QUALITY = property.CODE_QUALITY
        env.UNIT_TESTING = property.UNIT_TESTING
        env.CODE_COVERAGE = property.CODE_COVERAGE
	

}

def FAILED_STAGE
podTemplate(cloud:'openshift',label: 'docker',
  containers: [
    containerTemplate(
      name: 'jnlp',
      image: 'docker:dind',
      alwaysPullImage: true,
     // resourceRequestCpu: '50m',
     // resourceRequestMemory: '500Mi',
      workingDir: '/home/jenkins/agent',

      envVars: [envVar(key:'http_proxy',value:''),envVar(key:'https_proxy',value:'')],
      args: '${computer.jnlpmac} ${computer.name}',
      ttyEnabled: true
    )],volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),hostPathVolume(hostPath: '/etc/docker/daemon.json', mountPath: '/etc/docker/daemon.json')] )
{
 


node
{
    def MAVEN_HOME = tool "MY_MAVEN"
    def JAVA_HOME = tool "MY_JDK"
    env.PATH= "${env.PATH}${MAVEN_HOME}\\bin;${JAVA_HOME}\\bin"
    
    stage('Checkout')
    {
        readProperties()
        checkout([$class: 'GitSCM', branches: [[name: "*/${BRANCH}"]], doGenerateSubmoduleConfigurations: false, extensions:[], submoduleCfg: [], userRemoteConfigs: [[url: "${GIT_SOURCE_URL}"]]])
    }
    stage('Initial setup')
    {
       
        bat 'mvn clean'
    }
    if (env.UNIT_TESTING == 'True')
    {
	    try {
		    stage('Unit testing') {
			    			
        					bat 'mvn test'
			    			def defid = bat(script: 'git log --oneline -1',returnStdout: true).trim()
			    			def id = defid.substring(defid.lastIndexOf(" ")+1)
			    			echo id
			    			bat """curl --header "zsessionid":"_7cIVFUMTAe5YRxqNYHuc7obb0aBlXM1WYurWU8" -H "Content-Type":"application/json" -d"{\\"Defect\\":{\\"Name\\":\\"Automated Defect fixed\\",\\"Severity\\": \\"Cosmetic\\", \\"Priority\\": \\"Resolve Immediately\\", \\"State\\": \\"Fixed\\",\\"Requirement\\": \\"446239621908\\", \\"Description\\":\\"Defect is fixed\\"}}" https://rally1.rallydev.com/slm/webservice/v2.0/defect/${id}"""
			    			
				}
	    } catch (e) {
		    
		    def jenkins = env.BUILD_URL
		    echo jenkins
		    
		    def url = bat(script: 'git log --oneline -1',returnStdout: true).trim()
		    def usno = url.substring(url.lastIndexOf("/")+1,url.indexOf("?"))
		    echo usno
		    
		    def response = bat(script: """curl -g --header "zsessionid":"_7cIVFUMTAe5YRxqNYHuc7obb0aBlXM1WYurWU8" -H "Content-Type":"application/json" -d"{\\"Defect\\":{\\"Name\\":\\"Automated Defect Logging\\",\\"Severity\\": \\"Cosmetic\\", \\"Priority\\": \\"Resolve Immediately\\", \\"State\\": \\"Open\\",\\"Requirement\\": \\"${usno}\\",\\"Description\\": \\"Unit test failed, check Jenkins console output here: ${jenkins}\\"}}" https://rally1.rallydev.com/slm/webservice/v2.0/defect/create""",returnStdout: true).trim()
		    //def response = bat(script: """curl --header "zsessionid":"_7cIVFUMTAe5YRxqNYHuc7obb0aBlXM1WYurWU8" -H "Content-Type":"application/json" -d"{\\"Defect\\":{\\"Name\\":\\"Automated Defect: US2020\\",\\"Severity\\": \\"Cosmetic\\", \\"Priority\\": \\"Resolve Immediately\\", \\"State\\": \\"Open\\",\\"Requirement\\": \\"${usno}\\", \\"Description\\":\\"Jenkins URL: http://localhost:9090/job/java/193/console\"}}" https://rally1.rallydev.com/slm/webservice/v2.0/Defect/create
		    //echo response
		    echo "index of create res"
		    def jsonString = response.substring(response.indexOf('CreateResult')-2,response.length())
		    echo jsonString
		    //def ind = response.indexOf('CreateResult')
		    //echo ind
		    
		    
		    def jsonObj = readJSON text: jsonString
		    bat "echo ${jsonObj.CreateResult.Object._ref}"
		    def defecturl = jsonObj.CreateResult.Object._ref
		    echo defecturl
		    //def url = "https://rally1.rallydev.com/#/detail/userstory/446239621908?fdp=true"
		    //def usno = url.substring(url.lastIndexOf("/")+1,url.indexOf("?"))
		    //echo usno
		    //def api = '"Attachment":{"Content":"https://rally1.rallydev.com/slm/webservice/v2.0/attachmentcontent/450673452248","Artifact":"+"${defecturl}"+","ContentType":"application/octet-stream","Name":"Jenkins_URL3.txt"}'
		  //  bat '''curl --header "zsessionid":"_7cIVFUMTAe5YRxqNYHuc7obb0aBlXM1WYurWU8" -H "Content-Type":"application/json" -d"{\"\${api}\"}" https://rally1.rallydev.com/slm/webservice/v2.0/attachment/create'''
		     bat """curl --header "zsessionid":"_7cIVFUMTAe5YRxqNYHuc7obb0aBlXM1WYurWU8" -H "Content-Type":"application/json"   -d"{\\"Attachment\\":{\\"Content\\":\\"https://rally1.rallydev.com/slm/webservice/v2.0/attachmentcontent/453343285796\\",\\"Artifact\\":\\"${defecturl}\\",\\"ContentType\\":\\"application/octet-stream\\",\\"Name\\":\\"Jenkins_URL3.txt\\"}}" https://rally1.rallydev.com/slm/webservice/v2.0/attachment/create"""
		     currentBuild.result = 'FAILURE'
		     return
	    }
    }
		    
	
    
    if (env.CODE_QUALITY == 'True')
    {
        stage('Code Quality')
        {
            echo 'quality test'
        }
    }
    if (env.CODE_COVERAGE == 'True')
    {
        stage('Coverage testing')
        {
           echo 'mvn cobertura:cobertura'
	   return
        }
    }
    if (env.SECURITY_TESTING == 'True')
    {
        stage('Security testing')
        {
            echo 'security'
        }
    }
    if (env.SONAR == 'True')
    {
        stage('sonar testing')
        {
           echo 'mvn sonar:sonar'
        }
    }

    stage('Package')
   {
//   		script {
//        withCredentials([
//            usernamePasswordd(credentialsId: 'DockerID', usernameVariable: 'username', passwordVariable: 'password')
//          ])
//        {  
   		
	  
}
}

}		


