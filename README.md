# lambda_layer
CloudFormation yaml script to provision a Linux EC2 Instance that publishes a Python Lambda Layer. 
In this article I intend to explain what a Lambda Layer is, the benefits of it, and lastly how to create a Layer for a Python Lambda function. 

**<u>Lambda Layer</u>** 

A Lambda Layer is an isolated .zip file that contains libraries, packages and/or application code that is shareable between your Lambda functions. A common use case for a Lambda Layer is, for example your company has two separate Python Web Scrapers. Both functions use pandas to do some data processing. Since they both use the same Python packages, the smart solution is to create a Lambda Layer, so both functions can access the layer. Below is a visual from AWS that shows a solution with Lambda Layers vs with out Lambda Layers.

![Layers](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/86hc1c4myw9jw9enwqtc.gif)

**<u>Benefits of Lambda Layers</u>**

Lambda Layers can be used to reduce the size of uploaded deployment archives and also to increase the speed of deployment.

**<u>Creating a Python Lambda Layer</u>**

We will create the layer using IaC (CloudFormation). I know you might be thinking that it might be an overkill to spin up an instance just to create a Lambda Layer. Why not just do it locally? However this solves 2 main problems which are 

1. AWS Lambda runs in a Linux machine

2. Any Python Packages added to the Layer needs to be compiled with correct architecture (Linux x86_64).

You can find the complete CloudFormation Stack on my [GitHub](https://github.com/dirtbag-ctrl/lambda_layer).

**<u>Create stack parameters </u>**

```

---

Parameters:

  InstanceName:

    Type: String

    Description: Enter the name of your instance.

    Default: LinuxInstance

```

The only parameter we will be using is for our instance name.

**<u>Create IAM Role and Instance Profile for our EC2 Instance to use.</u>**

```

# Instance Profile

  InstanceProfile:

    Type: AWS::IAM::InstanceProfile

    Properties:

      InstanceProfileName: !Join ["_", [!Ref InstanceName, "Profile"]]

      Path: /

      Roles:

        - !Ref InstanceRole

  # Instance Role

  InstanceRole:

    Type: AWS::IAM::Role

    Properties:

      RoleName: !Join ["_", [!Ref InstanceName, "EC2PublishLambdaLayer"]]

      AssumeRolePolicyDocument:

        Version: 2012-10-17

        Statement:

          - Effect: Allow

            Principal:

              Service:

                - ec2.amazonaws.com

            Action:

              - sts:AssumeRole

      Path: /

      Policies:

        - PolicyName: !Join ["_", [!Ref InstanceName, "Policy"]]

          PolicyDocument:

            Version: "2012-10-17"

            Statement:

              - Effect: Allow

                Action: "lambda:PublishLayerVersion"

                Resource: "*"

```

**NOTE**: The Policy Document contains the action lambda:PublishLayerVersion, without this policy we will not be able to publish the Lambda Layer from within the EC2 Instance.

**<u>Create EC2 Instance</u>**

```

 # EC2 Instance

  Instance:

    Type: AWS::EC2::Instance

    Properties:

      ImageId: ami-09d3b3274b6c5d4aa # Static AMI ID chosen from List of AMIs in the console

      InstanceType: t2.micro

      IamInstanceProfile: !Ref InstanceProfile

      SecurityGroups:

        - !Ref SSHSecurityGroup

      # Install Python, Pandas and Publish Lambda Layer

      UserData:

        Fn::Base64: |

          #!/bin/bash 

          sudo amazon-linux-extras install python3.8

          curl -O https://bootstrap.pypa.io/get-pip.py

          python3.8 get-pip.py --user

          mkdir -p home/ec2-user/python

          cd home/ec2-user

          python3.8 -m pip install pandas -t python/

          zip -r layer.zip python

          aws lambda publish-layer-version --layer-name pandas-layer --zip-file fileb://layer.zip --compatible-runtimes python3.8 --region us-east-1

      Tags:

        - Key: Name

          Value: !Ref InstanceName

```

If you look at the UserData property, we are passing bash scripts. This is where the magic happens. We defined scripts to install Python and pandas. We then zip the directory that has the Python Package and lastly publish the Lambda Layer to AWS. 

**<u>Upload CF Stack</u>**

To upload your CF stack, use the command below.

```

aws cloudformation deploy 

--template-file <path_to_file> 

--stack-name <stack_name> 

--region us-east-1 

--capabilities CAPABILITY_NAMED_IAM

```

If the above command is successful, you will get an output on your terminal saying "Successfully created/updated stack - <stack_name> ". The stack should take a few minutes to provision our EC2 Instance and execute our script. Lastly, navigate to Lambda Layers in the console and you should see the Layer you created. 

![Lamda Layer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8gi8nzoy9xsolwaij17z.png)

**<u>Clean Up</u>**

To clean up all resources that were created simply delete the CloudFormation stack. 

```

aws cloudformation delete-stack 

--stack-name <stack_name>

```

