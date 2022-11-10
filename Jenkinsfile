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
                    echo "Hello! I am executing shell"
                    aws sts assume-role --role-arn arn:aws:iam::507216733449:role/Packer --role-session-name "kops-ami" --duration-seconds 3600
                    '''
                }
            }
        }

    }
}
