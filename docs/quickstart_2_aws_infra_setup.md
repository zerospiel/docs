# QuickStart 2 - AWS infrastructure setup

For this next phase of the k0rdent QuickStart, you’ll need the `clusterawsadm` tool (we installed this in [QuickStart 10()) and an AWS account with administrator permissions.

If you already have an access key and secret for your admin account, you can move on to . Otherwise, let’s go ahead and get them.

Start by logging into your admin account and click your name in the upper right-hand corner, then go to Security Credentials and scroll down to Access Keys. Click Create Access Key.

Check the box next to “I understand creating a root access key is not a best practice, but I still want to create one.” and click Create Access Key.

Save the access key and secret somewhere safe; you won’t be able to access the secret once you leave this page.

Now set your environment variables with the key and secret you just created:

```shell
export AWS_ACCESS_KEY_ID=EXAMPLE_ACCESS_KEY_ID
export AWS_SECRET_ACCESS_KEY=EXAMPLE_SECRET_ACCESS_KEY
export AWS_SESSION_TOKEN=EXAMPLE_SESSION_TOKEN # Optional. If you are using Multi-Factor Auth.
```

Now you’ll need to create the Cloudformation stack that k0rdent will use to interact with AWS. You can do this with the clusterawsadm tool:

```shell
clusterawsadm bootstrap iam create-cloudformation-stack
```

This stack creates Roles and Policies we’re going to need later. This is an example of the output:

```shell
Attempting to create AWS CloudFormation stack cluster-api-provider-aws-sigs-k8s-io

Following resources are in the stack:

Resource                  |Type                                                                                  |Status
AWS::IAM::InstanceProfile |control-plane.cluster-api-provider-aws.sigs.k8s.io                                    |CREATE_COMPLETE
AWS::IAM::InstanceProfile |controllers.cluster-api-provider-aws.sigs.k8s.io                                      |CREATE_COMPLETE
AWS::IAM::InstanceProfile |nodes.cluster-api-provider-aws.sigs.k8s.io                                            |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::743175908171:policy/control-plane.cluster-api-provider-aws.sigs.k8s.io   |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::743175908171:policy/nodes.cluster-api-provider-aws.sigs.k8s.io           |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::743175908171:policy/controllers.cluster-api-provider-aws.sigs.k8s.io     |CREATE_COMPLETE
AWS::IAM::ManagedPolicy   |arn:aws:iam::743175908171:policy/controllers-eks.cluster-api-provider-aws.sigs.k8s.io |CREATE_COMPLETE
AWS::IAM::Role            |control-plane.cluster-api-provider-aws.sigs.k8s.io                                    |CREATE_COMPLETE
AWS::IAM::Role            |controllers.cluster-api-provider-aws.sigs.k8s.io                                      |CREATE_COMPLETE
AWS::IAM::Role            |eks-controlplane.cluster-api-provider-aws.sigs.k8s.io                                 |CREATE_COMPLETE
AWS::IAM::Role            |nodes.cluster-api-provider-aws.sigs.k8s.io                                            |CREATE_COMPLETE
```

Now let’s go ahead and create the user that will actually take the actions we’ll need to take. Go to the console at https://aws.amazon.com and sign in. From there, follow these steps:

* Click your name in the upper-right corner.
* Click Credentials.
* On the left, click Users->Create New User.
* Enter a name for the user, then click Next.
* Click Attach Policies Directly and use the search box to add the following policies:
    ```
    control-plane.cluster-api-provider-aws.sigs.k8s.io
    controllers.cluster-api-provider-aws.sigs.k8s.io
    nodes.cluster-api-provider-aws.sigs.k8s.io
    ```
Note: if you’re using the search, you may have to add these one at a time, clicking Next to move on, then Previous to come back and add additional policies.

* Once you’ve added all three policies, click Create User.
* Click the user.
* Click the Security Credentials tab.
* Scroll down to Access Keys and click Create Access Key.
* Choose Command Line Interface (CLI) and the “I understand the above recommendation and want to proceed to create an access key.” checkbox, then click Next, then Create Access Key.
* Click Download .CSV file so you don’t lose the secret key.  

Now we can finally install the Credentials into k0rdent.  Go ahead and set the environment variables to match the access key and secret that you just created:

```shell
export AWS_REGION=us-east-2
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_ACCESS_KEY"
```

Now go ahead and create the credentials objects:

```shell
make setup-aws-creds
```

This command creates the Credential objects that templates will reference so that they can interact with the AWS infrastructure.

It takes a few seconds for the credentials to be completely ready, so check on the status with:

```shell
PATH=$PATH:./bin kubectl -n hmc-system get credentials aws-cluster-identity-cred
```

When they’re ready, you’ll see the READY state is true, as in:

```shell
NAME                        READY   DESCRIPTION
aws-cluster-identity-cred   true    Basic AWS credentials
```

Now we’re ready to go ahead and start working with k0rdent to [deploy a managed cluster on AWS](quickstart_3_deploy_managed_cluster_aws.md).


