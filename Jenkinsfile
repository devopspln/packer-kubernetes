properties(
        [buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '14')),[$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false],
        parameters([
                    string(defaultValue: "ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-*", description: 'Official kops image filter?', name: 'BASE_AMI_FILTER'),
                    string(defaultValue: "k8s-1.18-ubuntu-focal-20.04-amd64-hvm-ebs", description: 'KOPS image prefix? (leave default unless Amazon change naming convention)', name: 'PREFIX'),
                    string(defaultValue: "mds-ccc", description: 'KOPS image postfix?', name: 'POSTFIX'),
                    string(defaultValue: "ubuntu", description: 'SSH User on Base AMI (Debian: admin, Ubuntu: ubuntu)?', name: 'BASE_AMI_USER'),
                    password(defaultValue: 'SECRET', description: 'enter temporary AWS_ACCESS_KEY_ID', name: 'AWS_ACCESS_KEY_ID'),
                    password(defaultValue: 'SECRET', description: 'enter temporary AWS_SECRET_ACCESS_KEY', name: 'AWS_SECRET_ACCESS_KEY'),
                    password(defaultValue: 'SECRET', description: 'enter temporary AWS_SESSION_TOKEN', name: 'AWS_SESSION_TOKEN'),
                    booleanParam(name: 'APPARMOR', defaultValue: false, description: 'install AppArmor?'),
            ])
        ])

        String nodeLabel = UUID.randomUUID().toString()
        echo "Unique node label ${nodeLabel}"

        podTemplate(label: nodeLabel, annotations: [podAnnotation(key: "iam.amazonaws.com/role", value: "node1")], serviceAccount: 'jenkins-agents-serviceaccount', containers: [
        containerTemplate(name: 'packer', image: 'image', ttyEnabled: true, command: 'cat')
    ]) {

        node(nodeLabel) {
        wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'XTerm']) {
            stage('Preparing Kops AMI') {
                container('packer') {
                    checkout scm
                    sh ('''#!/bin/bash
                    set -o pipefail
                    export TIMESTAMP=$(date +%F)
                    cat <<EOF >>packer.json
{
  "builders": [{
    "type": "amazon-ebs",
    "assume_role": {
        "role_arn": "arn:aws:iam::507216733449:role/Packer",
        "policy_arns": "arn:aws:iam::507216733449:policy/assumerole"
    },
    "ami_name": "$PREFIX-$TIMESTAMP-$POSTFIX",
    "region": "eu-central-1",
    "source_ami_filter": {
    "filters": {
      "virtualization-type": "hvm",
      "name": "$BASE_AMI_FILTER",
      "root-device-type": "ebs"
    },
    "owners": ["507216733449"],
    "most_recent": true
    },
    "instance_type": "t2.small",
    "ssh_username": "$BASE_AMI_USER",
    "ami_description": "Source AMI: {{ .SourceAMI }} - {{ .SourceAMIName }}",
    "vpc_filter": {
        "filters": {
        "tag:Name": "AWS Batch VPC",
        "isDefault": "false"
        }
    },
    "subnet_filter": {
        "filters": {
          "tag:Name": "Subnet-Public-1"
        },
        "most_free": true,
        "random": false
      },
    "temporary_security_group_source_cidrs": ["52.94.248.112/28", "52.95.255.128/28", "52.58.0.0/15", "52.28.0.0/16", "52.57.0.0/16", "52.29.0.0/16", "54.93.0.0/16", "18.194.0.0/15", "35.156.0.0/14", "18.153.0.0/16", "18.196.0.0/15", "18.184.0.0/15", "3.120.0.0/14", "3.124.0.0/14", "52.95.248.0/24", "52.46.184.0/22", "10.0.0.0/16"]
  }],
"provisioners": [
        {
            "type": "ansible",
            "playbook_file": "${WORKSPACE}/nts_base_ami_jobs/kops_base_ami/playbook.yaml",
            "extra_arguments": ["-e", "APPARMOR=$APPARMOR"],
            "user": "me"
        }
    ]
}
EOF
                        cat packer.json
                        packer build -debug -machine-readable -var "aws_access_key=$AWS_ACCESS_KEY_ID" -var "aws_secret_key=$AWS_SECRET_ACCESS_KEY" -var "token=$AWS_SESSION_TOKEN" packer.json | tee packer.log
                        if [ $? -eq 0 ]
                        then
                            packer_ami_id=$(cat packer.log | egrep "artifact,0,id" | rev | cut -f1 -d, | rev |cut -f 2 -d ":")
                            echo "\nNewly created AMI ID: $packer_ami_id"
                            echo "\nPlease remember to spread it all AWS Accounts with lambda: share_and_encrypt from account AWS-0004!"
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
}
