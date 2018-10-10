# Lab 1: Cloudformation and S3

Create and configure an S3 bucket using cloudformation templates.

## Lab 1.1: CloudFormation Template Requirements
Create the most minimal CFN template possible that can be used to create an AWS Simple Storage Service (S3) Bucket.
- Always write your CloudFormation [templates in YAML](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-formats.html).
- Launch a Stack by using the [AWS CLI tool](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html) to run the template. Use your preferred region.

### Solution to 1.1:
Yml template:
```
AWSTemplateFormatVersion: "2010-09-09"
  Description: "Brian S3 Bucket"
	Resources:
  	  BrianS3Bucket:
    	Type: "AWS::S3::Bucket"
    	Properties:
      	  BucketName: "brian-bucket-asdfxyz1"
```

Command line to create the stack and S3 bucket:
```
aws cloudformation create-stack --stack-name brians3stack --template-body file://s3_template.yml
```

## Lab 1.2: Stack Parameters
Update the same template by adding a CloudFormation [Parameter](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) to the stack and use the parameter’s value as the name of the S3 bucket.
- Put your parameter into a separate JSON file and pass that file to the CLI.
- Update your stack.

### Solution to 1.2:
Yml template:
```
AWSTemplateFormatVersion: "2010-09-09"
Description: "Brian S3 Bucket"
Parameters:
  BrianS3BucketName:
    Type: String
Resources:
  BrianS3Bucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: 
        Ref: BrianS3BucketName
```

Json file containing the parameter value:
```
[
  {
    "ParameterKey": "BrianS3BucketName",
    "ParameterValue": "brian-s3-bucket-2018-09-28-xyz1"
  }
]
```

Command line to create the stack and S3 bucket using the parameter:
```
aws cloudformation create-stack --stack-name brians3stack --template-body file://s3_template.yml --parameters file://bucket.json
```

## Lab 1.3: Pseudo-Parameters
Update the same template by prefixing the name of the bucket with the Account ID in which it is being created, no matter which account you’re running the template from (i.e., using [pseudo-parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html)).
- Use built-in CFN string functions to combine the two strings for the Bucket name.
- Do not hard code the Account ID. Do not use an additional parameter to provide the Account ID value.
- Update the stack.

### Solution to 1.3:
Yml file:
```
AWSTemplateFormatVersion: "2010-09-09"
Description: "Brian S3 Bucket"
Parameters:
  BrianS3BucketName:
    Type: String
Resources:
  BrianS3Bucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: !Join [ '-', [Ref: "AWS::AccountId", Ref: BrianS3BucketName] ]
```

Json file and cli commands are the same as in lab 1.2


## Lab 1.4: Using Conditions
Update the same template one final time. This time, use a CloudFormation [Condition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html) to prefix the name of the bucket with the region name of your preferred region or the Account ID if the current region of execution is any other region.
- Update the stack that you originally deployed.
- Create a new stack with the same stack name, but this time deploying to some region other than your preferred region.

### Solution to 1.4:
Yml file:
```
AWSTemplateFormatVersion: "2010-09-09"
Description: "Brian S3 Bucket"
Parameters:
  BrianS3BucketName:
    Type: String
Conditions:
  PreferredRegion: !Equals ["AWS::Region", "us-east-1"]
Resources:
  BrianS3Bucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName:
        !If
          - PreferredRegion
          - !Join [ '-', [Ref: "AWS::Region", Ref: BrianS3BucketName]]
          - !Join [ '-', [Ref: "AWS::AccountId", Ref: BrianS3BucketName]]
```

Command line:
```
aws cloudformation create-stack --stack-name brians3stack --template-body file://s3_template_1.4.yml --parameters file://bucket.json
```

## Lab 1.5: Termination Protection; Clean up

- Before deleting this lesson’s Stacks, apply Termination Protection to one of them.
- Try to delete the Stack using the AWS CLI. What happens?
- Remove termination protection and try again.
- List the S3 buckets in both regions once this lesson’s Stacks have been deleted to ensure their removal.