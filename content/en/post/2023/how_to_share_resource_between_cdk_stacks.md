+++
title = "How to share resource between CDK stacks"
date = 2023-06-28T09:41:00-07:00
lastmod = 2023-06-28T15:52:51-07:00
tags = ["aws"]
categories = ["aws"]
draft = false
toc = true
+++

## <span class="section-num">1</span> Introduction {#introduction}


### <span class="section-num">1.1</span> IaC {#iac}

Infrastructure as code(IaC) is the managing and provisioning of infrastructure through code instead of manual processes, for example, clicking button, adding or editing roles in AWS console.


### <span class="section-num">1.2</span> AWS CloudFormation {#aws-cloudformation}

AWS CloudFormation is the original IaC tool for AWS, released in 2011, which uses template files to automate and mange the setup of AWS resources.


### <span class="section-num">1.3</span> AWS CDK {#aws-cdk}

AWS Cloud Development Kit(CDK) is a product provided by AWS that makes it easier for developers to manage their infrastructure with familiar programming languages like TypeScript, Python, Java, etc.

And, CDK is standing on the shoulder of Cloudformation, providing tools for developers by leveraging Cloudformation.

A stack is a collection of AWS resources that you can manage as a single unit, like a box.

For instance, this box could include all the resources required to run an application or Lambda service, such as S3 Buckets (storage), Roles (authorization), Lambda Function (computing), API Gateway (access point), Alarm, Monitoring, etc.


## <span class="section-num">2</span> Problem {#problem}

I am currently working on a project which requires to set up two stacks, one stack( `GlueStack` ) for defining a list of AWS Glue tables and the other stack( `ServiceStack` ) for definition of Lambda service and associated resources.

{{< figure src="/ox-hugo/two_stacks.jpg" >}}

In fact, S3 bucket names have to be globally unique within a partition, which means crossing the whole AWS customer base.

You are unable to create a S3 bucket with bucket name which is in use by another AWS customer or your own account.

So it's safer to let CloudFormation generate a random bucket name for a developer when he need to initialize a S3 bucket.

However, there is new a problem I face: since the S3 bucket name is randomly generated characters, if `GlueStack` need to read the bucket created by `ServiceStack`, how could I share the bucket name between two stacks?

While these two stacks are isolated and separated, resources collection.

{{< figure src="/ox-hugo/s3_bucket_problem.jpg" >}}


## <span class="section-num">3</span> Solution {#solution}

Fortunately, CDK offers a facility named `CfnOutput` to export a deployed resource, so that the consumer of the resource is able to `Import` required resource.

{{< figure src="/ox-hugo/export.jpg" >}}

1.  Define the required resource in `ServiceStack` (producer), for instance, a S3 bucket:
    ```javascript
    import { Bucket } from 'aws-cdk-lib/aws-s3';

    const s3Bucket = new Bucket(this, 'MyBucketId', {});
    ```
2.  Export the resource by specifying the `value` and `exportName`:
    ```javascript
    import { CfnOutput } from 'aws-cdk-lib';

    // export the generated bucket name to other stack
    new CfnOutput(this, 'exportRequiredS3Bucket', {
        value: s3Bucket.bucketName,
        exportName: 'exportRequiredS3Bucket',
    });
    ```
3.  Import the required resource in `GlueStack` (consumer):
    ```javascript
    import { Fn} from 'aws-cdk-lib';

    const requiredS3BucketName = Fn.importValue('exportRequiredS3Bucket');
    ```

If we take a closer look at the synthesized CFN template for ServiceStack, we could find:

```json
"Outputs": {
    "exportRequiredS3Bucket": {
        "Value": {
            "Ref": "MyBucketId737FC949"
        },
        "Export": {
            "Name": "exportRequiredS3Bucket"
        }
    }
```

The synthesized CFN template for `GlueStack`:

```json
{
    "Fn::ImportValue": "exportRequiredS3Bucket"
}
```

This is the way about how to share value between two stacks.


## <span class="section-num">4</span> Reference {#reference}

-   [API Document: class CfnOutput (construct)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnOutput.html)
-   [AWS Cloud Development Kit (AWS CDK) v2](https://docs.aws.amazon.com/cdk/v2/guide/stacks.html)
