# tap-ts-starter #

This is a [Singer](https://singer.io) tap built with TypeScript/javascript that runs in Node and produces JSON-formatted data following the [Singer spec](https://github.com/singer-io/getting-started/blob/master/docs/SPEC.md#singer-specification), and most of the spec is reflected in [tap-types.ts](./src/tap-types.ts).

This tap:
- Scans a local folder, treating the files it finds there as emails (MIME), parsing them into JSON with [Nodemailer.Mailparser](https://nodemailer.com/extras/mailparser/)
- Outputs a schema along with the resulting json for each file

This tap is also meant as a template to be forked for other uses. It separates the scanning of a resource collection (e.g. a folder) and the parsing of the individual resources (e.g. MIME files) into separate modules for easy drop-in replacement. A scanner module is included (scan-dir.ts for scanning local folders) and a parser module (parse-mime.ts for parsing emails) is included as well.

This code path is documented [here](https://rawgit.com/donpedro/tap-ts-starter/master/dist/docs-tap/index.html).

### New-School Code
If you're used to JavaScript code, here are a few newer ES6/ES7/TypeScript code features we use that might be new to you:
* [Arrow functions](https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/) are largely interchangable
 with the more familiar ```function``` syntax:

 ```let aFunction = () => {...```

 is roughly equal to

 ```function aFunction() {...```
* [Promises](https://scotch.io/tutorials/javascript-promises-for-dummies) replace callbacks to clean up and clarify our code
* [Async/await](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9) builds on promises to make asynchronous code almost as simple (in many cases) as synchronous.

### AWS Lambda Deployment
In addition to running as Singer taps, parsers can also be deployed as AWS Lambda functions. This allows you to take the exact same parser that your tap uses and deploy it to parse files one-at-at-time via triggers which run as they are dropped into a bucket. This functionality is enabled out of the box; the deploy script will create the bucket, deploy the parser as a Lambda function and add a trigger to call it when files are created in the bucket.

This code path is documented [here](https://rawgit.com/donpedro/tap-ts-starter/master/dist/docs-aws/index.html).

### Quick Start
* Dependencies: 
    * [git](https://git-scm.com/downloads)
    * [nodejs](https://nodejs.org/en/download/releases/) - At least v6.3 (6.9 for Windows) required for TypeScript debugging
    * npm (installs with Node)
    * typescript - installed as a development dependency
    * serverless - `npm install -g serverless` to install globally
* Clone: `git clone https://github.com/donpedro/tap-ts-starter.git`
    * After cloning the repo, be sure to run `npm install` to install npm packages
* Debug: with [VScode](https://code.visualstudio.com/download) use `Open Folder` to open the project folder, then hit F5 to debug. This runs without compiling to javascript using [ts-node](https://www.npmjs.com/package/ts-node)
* Test: `npm test` or `npm t`
* Compile documentation: `npm run build-docs-tap` and `npm run build-docs-aws`
* Compile to javascript: `npm run build`
* Deploy: `serverless deploy --aws-profile [profilename]`
    * depends on [aws-cli](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) [named profiles](http://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)
* More options are included from [TypeScript Library Starter](https://github.com/alexjoverm/typescript-library-starter.git) and are documented [here](starter-README.md)
* Run using included test data (be sure to build first): `node dist/tap-main.cjs.js --config testdata/emails.tap-config.json`

### Notes
Note: This document is written in [Markdown](https://daringfireball.net/projects/markdown/). We like to use [Typora](https://typora.io/) and [Markdown Preview Plus](https://chrome.google.com/webstore/detail/markdown-preview-plus/febilkbfcbhebfnokafefeacimjdckgl?hl=en-US) for our Markdown work.

### XML Parser

There is a typescript file called 'xml-parse.ts', this file exports a function 'parseItem' and this function returns a Promise. The Promise uses an npm module 'xml2js' to parse the given xml file to json. When the parsing is complete the Promise resolves if a valid is result is parsed or rejects if an error is encountered. 

npm module: [xml2js](https://www.npmjs.com/package/xml2js )

### AWS Deployment

Here is a guide to deploy functions to AWS Lambda, to use buckets with AWS S3, and the starter [project](https://github.com/donpedro/tap-ts-starter). 

I followed this [guide](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) for configuring the aws-cli

Download and Install "Windows x86-64 executable installer" for [Python](https://www.python.org/downloads/release/python-365/) 

- Add "C:\Python36" and "C:\Python36\Scripts" to PATH environment variable
- Add "C:\Users\\\[user]\AppData\Roaming\Python\Python36\Scripts" to PATH env var
- Run command "pip install awscli"

AWS stores credentials in two files in folder "{userprofile}\\.aws"

- to add a named profile to credential folder use "aws configure --profile [a profile name]" command. You will then be prompted to enter access key, secret key, region, and output type

- To get this information 

  - Create your own AWS account (aws.amazon.com)
  - Find security credentials on the dropdown menu when you click your profile name
  - Go to users, add new user, select programmatic access
  - Create group and add the new user to the newly created group
  - Finish creating the user and it will give you the access and secret key, copy those keys to the cli after running "aws configure --profile [a profile name]", for region use "us-west-2" and for output press enter and it will default to json
  - Now you can look in the .aws/configure and .aws/credentials files and see that the profile was added

- You now have a named profile to run the "serverless deploy --aws-profile [profilename]" cmd, but it will still fail because you do not have 'permission' to use the AWS services. To fix:

  - Go to the IAM Management Console" for AWS

  - Go to policies, create policy, name it 'cloudformation', select json, and copy this into it:

    - ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "Stmt1449904348000",
                  "Effect": "Allow",
                  "Action": [
                      "cloudformation:CreateStack",
                      "cloudformation:CreateChangeSet",
                      "cloudformation:ListStacks",
                      "cloudformation:UpdateStack",
                      "cloudformation:DescribeChangeSet",
                      "cloudformation:ExecuteChangeSet"
      ,
      "cloudformation:DescribeStacks",
                      "cloudformation:DescribeStackResource",
                      "cloudformation:ValidateTemplate"            ],
                  "Resource": [
                      "*"
                  ]
              }
          ]
      }
      ```

  - Create another policy, name it FullAccess_APIGatewayRec" and copy this to it:

    - ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "Stmt1467321765000",
                  "Effect": "Allow",
                  "Action": [
                      "apigateway:*"
                  ],
                  "Resource": [
                      "*"
                  ]
              }
          ]
      }
      ```

  - Go to group that you created for your user, go to permissions, attach policy, and attach the following policies:

    - "cloudformation" //manually created
    - "FullAccess_APIGatewayRec" //manually created
    - "AWSLambdaFullAccess"
    - "IAMFullAccess"
    - "AmazonS3FullAccess"
    - "CloudWatchFullAccess"
    - "CloudFrontFullAccess"
    - "AWSCloudFormationReadOnlyAccess"

- Deployment:

  - In file "serverless.yml" change bucket name to something unique. Example: "fdsa-trigger-bucket12" in any place you see "fdsa-trigger-bucket"
  - In file "serverless.yml", under functions, for each function (filetrigger, hello), change handler to "dist/es/handler.[functionname]"
  - Navigate to project folder then run "npm run build" and then run "serverless deploy --aws-profile [profilename]" command
  - Go to AWS Lambda service, change region to oregon, select manage functions
  - There should be two functions which are also found in your "handler.ts" source file
  - One of these functions listens for an event from AWS S3, this event happens when a file is dropped into a specific bucket in S3.
    - Go to AWS S3 service
    - You should find a bucket called "fdsa-trigger-bucket", this is where you can drop a file that will be processed by the Lambda function.
    - Drop a file, Go to AWS CloudWatch, go to logs, go to the correct function, and you should see a log of the file that was dropped.

### Testing

We are using [Jest](https://facebook.github.io/jest/docs/en/getting-started.html) for our unit testing. 

So far `src/parse-mime.ts` is the only module that is being tested. 

In the folder `test` there is a file `parse-mime.test.ts` which is the Jest test case for testing the parser.

It works by first reading two files:

- One is the test data, in this case .eml files
- Second is a .json file that contains an expected result if we ran `src/parse-mime.ts` on the test file

These two files are passed into a "matcher" which is a jest function used to check that values meet a certain condition. We are checking if the expected result file matches what we actually get when we run `src/parse-mime.ts`.

#### To add a test case: 

- Add a .eml file to the `testdata/emails` folder
  - Run the VS Code debugger with the configuration `Debug mime parser` while your new test file is open on the screen.
  - Copy the output from the debug console
  - Run the copied result through this [JSON-validator](https://jsonlint.com/) in order to check is JSON is valid and to format in a more readable way
- Add a .json file to the `testdata/testoutput` folder
- Take the newly formatted JSON and paste it into your test output file
- In "email.test-config.json" there is an array of JSON objects. Each object has two properties: `testdata` and `expectedresult`. Add a new object with your new file names.
- The tester will now run through all tests including the newly added test case

To run the tester: run the command `npm test` 

As the tester runs it will print which files are being tested

Example: `Tested data input: test.eml with expected output: Otest.json`

If a test case fails, the files it failed with will be the last files printed. 



​	