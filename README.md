### Synchronously process your receipt and invoice images using Amazon Textract AnalyzeExpense


## Architecture

![architecture](architecture.png)

At a high level, the solution architecture includes the following steps:
1.	Sets up an Amazon S3 source and output buckets storing raw expense documents images in png, jpg(jpeg) formats.
2.	Configures an Event Rule based on event pattern in Amazon EventBridge to match incoming S3 PutObject events in the S3 folder containing the raw expense document images.
3.	Configured EventBridge Event Rule sends the event to an AWS Lambda function for futher analysis and processing.
4.	AWS Lambda function reads the images from Amazon S3, calls Amazon Textract AnalyzeExpense API, uses Amazon Textract Response Parser to de-serialize the JSON response and uses Amazon Textract PrettyPrinter to easily print the parsed response and stores the results back to Amazon S3 in different formats.

## Prerequisites
1. python
2. AWS CLI

## Deployment
1. Download this git repo on your local machine
2. Install Amazon Textract Pretty Printer and Response Parser
      ```
      cd amazon-textract-analyzeexpense

      python3 -m pip install --target=./ amazon-textract-response-parser

      python3 -m pip install --target=./ amazon-textract-prettyprinter
      ```
  3. Create lambda function deployment package (.zip) file
      ```
      zip -r archive.zip .
      ```
  4. Next, copy archive.zip file to a s3 bucket of your choice.
      ```
      aws s3 cp archive.zip s3://my-source-bucket
      ```
If you need to create a new bucket in us-east-1
>`aws s3api create-bucket --bucket my-source-bucket --region us-east-1`
>
And, for all other regions, add `--create-bucket-configuration` option
>`aws s3api create-bucket --bucket my-source-bucket --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2`

5. Open template.yaml file, replace s3 bucket name under "EventConsumerFunction:" section with your s3 bucket name, and save.
6. Finally, deploy this solution
```
aws cloudformation deploy --template-file template.yaml --stack-name myTextractAnalyzeExpense --capabilities CAPABILITY_NAMED_IAM
```


## Test
In order to test the solution, upload the receipts/invoice images in the Amazon S3 bucket created by CloudFormation template. This will trigger an event to invoke AWS Lambda function which will call the Amazon Textract AnalyzeExpense API, parse the response JSON, convert the parsed response into csv format and store it back to the same Amazon S3 Bucket. You can extend the provided AWS Lambda function further based on your requirements and also change the output format to other types like tsv, grid, latex and many more by setting the appropriate value of output_type when calling get_string method of textractprettyprinter.t_pretty_print_expense in Amazon Textract PrettyPrinter.

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.
