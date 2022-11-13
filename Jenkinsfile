String nodeLabel = UUID.randomUUID().toString()
echo "Unique node label ${nodeLabel}"
podTemplate(label: nodeLabel, annotations: [podAnnotation(key: "iam.amazonaws.com/role", value: "ip-172-20-33-161.ec2.internal")], serviceAccount: 'jenkins', containers: [
        containerTemplate(name: 'packer', image: 'devopspln/kops:v4', ttyEnabled: true, command: 'cat')
    ]) {

        node(nodeLabel) {
            stage('Preparing Kops AMI') {
                container('packer') {
                    checkout scm
                    sh ('''#!/bin/bash
                    set -o pipefail     
                    export TIMESTAMP=$(date +%F)
                    caller =`aws sts get-caller-identity`
                    echo $caller
                    cat <<EOF >>packer2.json.pkr.hcl

  source "amazon-ebs" "autogenerated_1" {
  ami_name                    = "packer"
  associate_public_ip_address = "true"
  instance_type        = "t2.micro"
  region               = "us-east-1"
  source_ami           = "ami-0149b2da6ceec4bb0"
  ssh_timeout          = "15m"
  ssh_username         = "ubuntu"
  subnet_id            = "subnet-01dd7bda0f00e17ee"
  vpc_id               = "vpc-0168e152250e71700"
}

build {
  sources = ["source.amazon-ebs.autogenerated_1"]

  provisioner "shell" {
    inline = [
        "echo 'foo'"
        ]
  }
}
EOF
                        cat packer2.json.pkr.hcl
                        packer build -debug -machine-readable packer2.json.pkr.hcl | tee packer.log
                        if [ $? -eq 0 ]
                        then
                            packer_ami_id=$(cat packer.log | egrep "artifact,0,id" | rev | cut -f1 -d, | rev |cut -f 2 -d ":")
                            echo "\nNewly created AMI ID: $packer_ami_id"
                            echo "\nPlease remember to spread it all AWS Accounts with lambda: share_and_encrypt from account !"
                            exit 0
                        else
                            echo "\n!!! Error during Packer execution !!!"
                            exit 1
                        fi
                    ''')

                }
            }       
    }
}
