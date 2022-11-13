podTemplate(containers: [
    containerTemplate(
        name: 'packer', 
        image: 'devopspln/kops:v3'
        )
  ]) {

    node(POD_LABEL) {
        stage('Packer') {
            container('packer') {
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
