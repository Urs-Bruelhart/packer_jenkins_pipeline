/***********************************************************************
                         Aricent Technologies Proprietary
 
This source code is the sole property of Aricent Technologies. Any form of utilization
of this source code in whole or in part is  prohibited without  written consent from
Aricent Technologies
 
File Name			  :Jenkinsfile
Principal Author	  :PRAVEEN KUMAR KRISHNAMOORTHY
Subsystem Name        :Jenkins Pipeline Study
Module Name           :
Date of First Release : Feb 25, 2017
Author                :PRAVEEN KUMAR KRISHNAMOORTHY
Description           :This is a sample program to study Continous Integration
Version               :1.0
Date(DD/MM/YYYY)      :Feb 25, 2017
Modified by           : PRAVEEN KUMAR KRISHNAMOORTHY
Description of change :Forked From 
 
***********************************************************************/
//TODO - Make SVN and GIT Checkout steps perfect with Jenkins way. Do not use Shell way.

//-----------------VARIABLE DEFINITIONS-----------------------
//def nexusRepoHostPort = nexusRepositoryHost    /*(NEEDS TO BE DEFINED)*/
//def nexusRepo = nexusRepository                /*(NEEDS TO BE DEFINED)*/
def nexusRepoHostPort = 172.19.74.230:8081    
def nexusRepo = MEC                
//------------------------------------------------------------

node {
  echo "Parameter List"
  echo "SCM Type    : ${scmSourceRepo}"
  echo "SCM Path    : ${scmPath}"
  echo "SCM User    : ${scmUsername}"
  echo "SCM Pass    : ${scmPassword}"
  echo "HTTP Proxy  : ${httpProxy}"
  echo "HTTPS Proxy : ${httpsProxy}"
  echo "Nexus Host & Port  :${nexusRepositoryHost}"  /*(NEEDS TO BE DEFINED)*/
  echo "Nexus Repo Name    :${nexusRepository}"      /*(NEEDS TO BE DEFINED)*/

// ---- Source Shell
// REMOVE THIS BLOCK IF INPUTS ARE TAKEN FROM NODE
  sh "export OS_PROJECT_NAME=admin"
  sh "export OS_USERNAME=admin"
  sh "export OS_PASSWORD=abc123"
  sh "export OS_AUTH_URL=http://172.19.74.169:35357/v2"
  sh "export OS_IDENTITY_API_VERSION=2"
  sh "export OS_IMAGE_API_VERSION=2"
//------------------------------------------------
/** ENABLE THIS IF INPUTS ARE TAKEN FROM NODE
  sh "export OS_PROJECT_NAME=${PROJECT_NAME}"
  sh "export OS_USERNAME=${USERNAME}"
  sh "export OS_PASSWORD=${PASSWORD}"
  sh "export OS_AUTH_URL=${AUTH_URL}"
  sh "export OS_IDENTITY_API_VERSION=${IDENTITY_API_VERSION}"
  sh "export OS_IMAGE_API_VERSION=${IMAGE_API_VERSION}"
**/

//------------------------------------------------
//To escape all Special Charecters in a given input string Username
  def pwdstr = scmPassword
  def usrstr = scmUsername
  scmPassword = pwdstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  scmUsername = usrstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
//To escape all Special Charecters in a given input string Username  
  def pwdstr2 = scmPassword
  def usrstr2 = scmUsername
  scmPassword = pwdstr2.replaceAll( /([@])/, '%40' )
  scmUsername = usrstr2.replaceAll( /([@])/, '%40' ) 
//----------------------------------------
  stage('Code Pickup')
  {
    echo "Source Code Repository Type : ${scmSourceRepo}"
    echo "Source Code Repository Path : ${scmPath}"
    
    if("${scmSourceRepo}".toUpperCase()=='SVN')
    {
       sh "svn co --username ${scmUsername} --password ${scmPassword} ${scmPath} ."
        
    }
    else if("${scmSourceRepo}".toUpperCase()=='GIT' || "${scmSourceRepo}".toUpperCase()=='GITHUB')
    {
      if(scmPath.startsWith("ssh://"))
        {
            scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
        } else
        {
            scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + ":" + scmPassword + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
        }
      echo "GIT PATH: ${scmPath}"
      try {
          //If we use git clone, it will not clone in the same path if we rebuild the pipeline
          sh 'ls -a | xargs rm -fr'
          } catch (error)
          {
          } 
      
      if(scmPath.startsWith("ssh://"))
          {
            if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0)
            {
              echo "Looks like this Jenkins behind Proxy"
              sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && sshpass -p ${scmPassword}   git clone ${scmPath} ."
            } else
            {
              echo "Looks like this Jenkins is not behind Proxy"
              sh "sshpass -p ${scmPassword}   git clone ${scmPath} ."
            }            
           } 
      else
           {
              if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0)
            {
              echo "Looks like this Jenkins behind Proxy"
              sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && git clone ${scmPath} ."
            } 
            else
            {
              echo "Looks like this Jenkins is not behind Proxy"
              sh "git clone ${scmPath} ."
            }            
           } 
    }
    else
    {
      error 'Unknown Source code repository. Only GIT and SVN are supported'
    }
  } 
//--------------------------------------  
// INITIALIZING
  stage ('Initialization')
{
def appModuleSeperated = fileExists 'app'
def testModuleSeperated = fileExists 'test'
def appPath = ''
def testPath = ''
if (appModuleSeperated) {
    echo 'App Module is found , assumed that application is present in /app directory'
    appPath='app/'
} else {
    echo 'There is no defined Application path , hence it is assumed that application is in current directory'
    appPath = ''
}

if (testModuleSeperated) {
    echo 'Test Module is found , assumed that Test Cases are Present for the concerned Modules and has to be performed'
    testPath = 'test/'
} else {
    echo 'No Test Modules found , hence it is assumed that no test environment and / or test cases to be performed'
    testPath = ''
}
  if (appPath + fileExists("${FileName}")) {
    echo "Packer file found at ${appPath}"
    PackerFile = appPath + "${FileName}"
} else {
    echo 'Packerfile not found under ' + appPath
  }
// COPYING APP Directory to Current Working Directory
  def appWorkingDir = (appPath=='') ? '.' : appPath.substring(0, appPath.length()-1)  

// NEXUS file for Time Stamp comparison. This file is used for comparing time stamps and differentiating input files from generated output files.        
//TODO - Tune it later. Dirty solution to identify the Jenkins generated artifacts for Nexus
        sh 'echo Nexus>Nexus.txt'
  //Dirty solution ends.
}
//END OF INITIALIZING.
//_______________________________________________________________________________________________________________________________________________________________________  
//BUILD & PACKING
  //---------------------------------------
  if("${stage}".toUpperCase() == 'BUILD') {
    echo 'The Requested Stage is Build Only,hence successful VM Images will be pushed to Temp Repo'
    stage('validate')
      {
        echo "Validating the template :${PackerFile}"
        sh "packer -v || packer validate ${PackerFile}"
      }
    stage('build')
      {
        echo "Building using packerfile :${PackerFile}"
        sh "packer build ${PackerFile}"
      }
    stage('test')
     {
// TESTS IF PRESENT COMES UNDER THIS SECTION
     }
  }
  else if("${stage}".toUpperCase() == 'CERTIFY') 
  {
    echo 'The Requested Stage is Certify,hence successful VM Images will be pushed to Temp Repo and provided with a sandbox for validation'
    stage('validate')
      {
        echo "Validating the template :${PackerFile}"
        sh "packer -v || packer validate ${PackerFile}"
      }
    stage('build')
      {
        echo "Building using packerfile :${PackerFile}"
        sh "packer build ${PackerFile}"
      }
    stage('test')
     {
// TESTS IF PRESENT COMES UNDER THIS SECTION
     }
    echo "VM Image Built and pushed into temp repository and provided with a sandbox"
  }  
  else if ("${stage}".toUpperCase() == 'DEPLOY')
  {
    echo 'The Requested Stage is deploy,hence successful VM Images will be pushed to Permanant Repository without validation (sandbox)'
    stage('validate')
      {
        echo "Validating the template :${PackerFile}"
        sh "packer -v || packer validate ${PackerFile}"
      }
    stage('build')
      {
        echo "Building using packerfile :${PackerFile}"
        sh "packer build ${PackerFile}"
      }
    stage('test')
     {
// TESTS IF PRESENT COMES UNDER THIS SECTION
     }
    echo "VM Image Built and pushed into openstack-glance repository"
  }
  
//END OF IMAGE PUSHING INTO REPOSITORY
// NEXUS UPDATE
  stage('Publish Jenkins Output to Nexus')
  {
  //TODO in code - Tune it later. Dirty solution to identify the Jenkins generated artifacts for Nexus. 
  //TODO in Jenkins - Needs a Credential with the name "Nexus"
  //TODO in Nexus web - Created a Hosted Site Repository with the name MEC
  //TODO - Nexus3 support - Check the Sonatype plugin once released
  //Nexus is a great component artifact repo. Does not look great dealing with binary documents and intermediate outputs.
  //The plugin is too weak. It can upload only one file and hence Zipped

    echo 'Publishing the artifacts...';
      //def PWD = pwd(); //"${PWD}/artifacts.tar.gz"
      //sh 'find . -type f -newer Nexus.txt -print0 | tar -czvf artifacts.tar.gz --ignore-failed-read --null -T -'
      //Fixed for archive overlap issue
      try{
        sh 'find . -type f -newer Nexus.txt -print0 | tar -zcvf artifacts.tar.gz --ignore-failed-read --null -T -' 
         } catch(Exception e) {
         }

  //Nexus 2
  //nexusArtifactUploader artifacts: [[artifactId: "${env.JOB_NAME}", classifier: '', file: 'artifacts.tar.gz', type: 'gzip']], credentialsId: 'Nexus', groupId: 'org.jenkins-ci.main.mec', nexusUrl: '13.55.146.108:8085/nexus', nexusVersion: 'nexus2', protocol: 'http', repository: 'MEC',version: "${env.BUILD_NUMBER}"
  //Nexus 3
  nexusArtifactUploader artifacts: [[artifactId: "${env.JOB_NAME}", classifier: '', file: 'artifacts.tar.gz', type: 'gzip']], credentialsId: 'Nexus', groupId: 'org.jenkins-ci.main.mec', nexusUrl: nexusRepoHostPort, nexusVersion: 'nexus3', protocol: 'http', repository: nexusRepo,version: "${env.BUILD_NUMBER}"
  sh 'rm Nexus.txt'    
  //Dirty solution ends  }
}
