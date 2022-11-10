podTemplate(containers: [
    containerTemplate(
        name: 'jnlp', 
        image: 'hashicorp/packer'
        )
  ]) {

    node(POD_LABEL) {
        stage('Get a Maven project') {
            container('jnlp') {
                stage('Shell Execution') {
                    sh '''
                    echo "Hello! I am executing shell
                    packer version
                    '''
                }
            }
        }

    }
}
