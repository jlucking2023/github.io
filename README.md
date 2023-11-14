# InsuranceLake Documentation

Acronyms
IL - InsuranceLake
3C’s - Collect, Cleanse, and Consume

IL Concepts
S3 Collect Bucket Database \ Table

QuickStart Install
Install the basics in 30 minutes
To set the region that InsuranceLake is installed in see the lib/configuration.py file.
https://gitlab.aws.dev/fsi-sat/aws-cdk-insurancelake-etl/-/blob/main/README.md
https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md

Full Install
can this be done both stand alone AND after the QuickStart is done?
install the basics + DevOps tools
Setup a new repository and deploy using CI/CD

Pull from an existing repository and deploy using CI/CD (will be the de-facto install once published in Github)

Install with 3 environments

Make a change and deploy automatically with self-mutating CodePipeline

InsuranceLake can be deployed with no VPC simply by removing the subnet definition in configuration.py. The VPC is only used if the customer needs it.

The public subnet is completely optional as well. InsuranceLake does not require any VPC, so it also does not require public subnets. Creating a VPC with half public subnets and half private is the default behavior. You can modify this by passing the subnet_configuration parameter to the VPC creation in lib/vpc_stack.py.

If the VPC is enabled in InsuranceLake, Glue is really the only service that will use it, and specifically, for Glue connections. If you try this out, you’ll see that the Glue connections specifically select the private subnet from the InsuranceLake-created VPC, through the vpc.subnets method.

1. Add Permission boundaries to all the roles that CDK creates example add Permission boundary name CloudCoreL3PermissionBoundary to all the roles 
2. Add the mandatory tags to all resources without which SCP will deny any resource creation.

