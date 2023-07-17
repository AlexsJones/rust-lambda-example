# rust-lambda-example

We are going to deploy a function called `zippy`

This is an example of how to build and deploy rust based lambda functions.

```
├── Cargo.lock
├── Cargo.toml
├── README.md
├── deploy # Typescript based deploy files for the stack
├── src # Rust lambda code
```


-----

#### Deploy

```
cdk deploy
```
#### Destroy
```
cdk destroy
```

This example is completed but the following steps are how to reproduce.

## Setup your own function 

_zippy is the placeholder name_


```bash
brew tap cargo-lambda/cargo-lambda
brew install cargo-lambda

cargo lambda new zippy

# setup local testing
rustup target add aarch64-unknown-linux-gnu
cargo lambda start 

# build artefact
cargo lambda build --release
npm install -g aws-cdk
mkdir deploy && cd deploy
cdk init app --language=typescript
# modify 
cat << EOF > deploy/lib/deploy-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as path from "path";
import {Code, Function, Runtime, FunctionUrlAuthType} from "aws-cdk-lib/aws-lambda";
import {CfnOutput} from "aws-cdk-lib";

export class DeployStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const handler = new Function(this, "zippy", {
      // The source code of your Lambda function. You can point to a
      // file in an Amazon Simple Storage Service (Amazon S3) bucket
      // or specify your source code as inline text.
      code: Code.fromAsset(path.join(__dirname, "..", "..", "target/lambda/zippy")),
      // The runtime environment for the Lambda function that you are uploading.
      // For valid values, see the Runtime property in the AWS Lambda Developer Guide.
      // Use Runtime.FROM_IMAGE when defining a function from a Docker image.
      runtime: Runtime.PROVIDED_AL2,
      handler: "does_not_matter",
      // The function execution time (in seconds) after which Lambda terminates the function.
      functionName: "zippy"
    });

    const fnUrl = handler.addFunctionUrl({
      authType: FunctionUrlAuthType.NONE,
    });

    new CfnOutput(this, 'TheUrl', {
      // The .url attributes will return the unique Function URL
      value: fnUrl.url,
    });
  }
}
EOF
# deploy with 
cdk synth
cdk deploy

# destroy
cdk destroy
```
