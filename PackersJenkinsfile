node {
  echo "Parameters"
  echo "SCM Type: ${scmSourceRepo}"
  echo "SCM Path: ${scmPath}"
  echo "SCM User: ${scmUsername}"
  echo "SCM Pass: ${scmPassword}"
  echo "HTTP Proxy: ${httpProxy}"
  echo "HTTPS Proxy: ${httpsProxy}"
  
  sh "export OS_PROJECT_DOMAIN_NAME=default"
  sh "export OS_USER_DOMAIN_NAME=default"
  sh "export OS_PROJECT_NAME=admin"
  sh "export OS_USERNAME=admin"
  sh "export OS_PASSWORD=password"
  sh "export OS_AUTH_URL=http://172.19.74.170:35357/v2"
//  sh "export OS_AUTH_URL=http://172.19.74.170/v3"
  sh "export OS_IDENTITY_API_VERSION=3"
  sh "export OS_IMAGE_API_VERSION=2"
  def err = null
  currentBuild.result = "SUCCESS"

  try {
    stage 'Checkout'
      checkout scm

    stage 'Validate'
      def packer_file = 'packer.json'
      print "Running packer validate on : ${packer_file}"
      /*sh export PATH=$PATH:~/packer/*/
      sh "packer validate ${packer_file}"

    stage 'Build'
      sh "packer build ${packer_file}"

    stage 'Test'
      print "Testing goes here."
  }

  catch (caughtError) {
    err = caughtError
    currentBuild.result = "FAILURE"
  }

  finally {
    /* Must re-throw exception to propagate error */
    if (err) {
      throw err
    }
  }
}
