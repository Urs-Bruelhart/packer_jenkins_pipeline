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
node {
  echo "Parameter List"
  echo "SCM Type    : ${scmSourceRepo}"
  echo "SCM Path    : ${scmPath}"
  echo "SCM User    : ${scmUsername}"
  echo "SCM Pass    : ${scmPassword}"
  echo "HTTP Proxy  : ${httpProxy}"
  echo "HTTPS Proxy : ${httpsProxy}"
  
// ---- Source Shell
  sh "export OS_PROJECT_NAME=admin"
  sh "export OS_USERNAME=admin"
  sh "export OS_PASSWORD=abc123"
  sh "export OS_AUTH_URL=http://172.19.74.169:35357/v2"
  sh "export OS_IDENTITY_API_VERSION=2"
  sh "export OS_IMAGE_API_VERSION=2"
  
//--------------------------------------
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
//BUILD & PACKAGE
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
    sh 'echo Nexus>Nexus.txt'
//END OF INITIALIZING.
//_______________________________________________________________________________________________________________________________________________________________________  
//BUILD & PACKING
  //---------------------------------------
  if("${stage}".toUpperCase() == 'VALIDATE') {
    echo 'It is inferred that the package is a validate only application'
    stage('VALIDATE')
      {
        echo "Running packer validate on : ${PackerFile}"
        echo "packer is being validated in" 
        sh "packer -v || packer validate ${PackerFile}"
      }
  }
  if("${stage}".toUpperCase() == 'BUILD') 
  {
    echo 'It is inferred that the package is a Build application , hence it has to be validated and built'
    stage('VALIDATE')
      {
        echo "Validating the template :${PackerFile}"
        sh "packer -v || packer validate ${PackerFile}"
      }
    stage('BUILD')
      {
        echo "Building using packerfile :${PackerFile}"
        sh "packer build ${PackerFile}"
      }
    echo "VM Image Built and pushed into openstack-glance repository"
  }  else if ("${stage}".toUpperCase() == 'TEST')
  {
    echo 'It is inferred that the package is a test application , hence it has to be moved to a provisioned with a runtime sandbox environment , validate , build and tested before pushing into repo'
    stage('VALIDATE')
      {
        echo "Validating the template :${PackerFile}"
        sh "packer -v || packer validate ${PackerFile}"
      }
    stage('BUILD')
      {
        echo "Building using packerfile :${PackerFile}"
        sh "packer build ${PackerFile}"
      }   
    stage('TEST')
     {
// TESTS IF PRESENT COMES UNDER THIS SECTION
     }
    echo "VM Image Built and pushed into openstack-glance repository"
  }
  
//END OF IMAGE PUSHING INTO REPOSITORY
// NEXUS UPDATE
  stage('Publish Jenkins Output to Nexus'){
        echo 'Publishing the artifacts...';
        sh 'rm Nexus.txt'
//NEXUS FLOW ENDS HERE
  }
}
