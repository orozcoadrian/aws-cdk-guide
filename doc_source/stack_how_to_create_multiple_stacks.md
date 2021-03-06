--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Create an App with Multiple Stacks<a name="stack_how_to_create_multiple_stacks"></a>

The following example shows one solution to parameterizing how you create a stack\. It creates one stack with a `t2.micro` AMI for development, and three stacks with the more expensive `c5.2xlarge` AMI\. The development stack and one of the production stacks use the same account to create the stack in `us-east-1`; the other two production stacks use different account IDs and regions\.

The file `MyApp-stack.ts` defines a property set that extends the `cdk.StackProps` class to add one additional property, `enc`, which specifies whether to set encryption on the Amazon S3 bucket in the stack\.

```
import cdk = require("@aws-cdk/cdk");
import s3 = require("@aws-cdk/aws-s3");

interface MyStackProps extends cdk.StackProps {
  enc: boolean;
}

export class MyStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyStackProps) {
    super(scope, id, props);

    if (props.enc) {
      new s3.Bucket(this, "MyGroovyBucket", {
        encryption: s3.BucketEncryption.KmsManaged
      });
    } else {
      new s3.Bucket(this, "MyGroovyBucket");
    }
  }
}
```

The file `MyApp.ts` creates two stacks\. One with an unencrypted bucket in the `us-west-2` region; the other with an encrypted bucket in the `us-east-1` region\.

```
import cdk = require("@aws-cdk/cdk");
import { MyStack } from "../lib/MyApp-stack";

const app = new cdk.App();

new MyStack(app, "MyWestCdkStack", {
  env: {
    region: "us-west-2"
  },
  enc: false
});

new MyStack(app, "MyEastCdkStack", {
  env: {
    region: "us-east-1"
  },
  enc: true
});
```

The following example shows how to deploy a stack with an encrypted bucket to the `us-east-1` region\.

```
cdk deploy MyEastCdkStack
```

If you look at the stack using cdk synth MyEastCdkStack, you should see a bucket similar to the following:

```
  MyGroovyBucketFD9882AC:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
```