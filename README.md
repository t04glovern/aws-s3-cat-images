# These Cats Don't Exist

Generating Cat images using StyleGAN

![img/catgen](img/catgen.png)

## Create Bucket

```bash
aws s3 mb s3://thesecatsdonotexist.com
```

## Copy in Files

```bash
aws s3 cp index.html s3://thesecatsdonotexist.com/index.html
```

## Generate the Cats

```bash
jupyter-notebook notebooks/catgen.ipynb
```

## AWS SageMaker Generate

**WARNING**: You will be paying > $2 per hour running this notebook. Ensure you delete it or turn it off when you aren't using it.

Create the SageMaker role that we'll attach to our SageMaker instance. Unfortunately since CloudFormation options for SageMaker do not allow us to attach Git repos as options yet.

```bash
aws cloudformation create-stack \
    --stack-name "cat-gen-sagemaker-role" \
    --template-body file://cloudformation/sagemaker_role.yaml \
    --capabilities CAPABILITY_IAM
```

Once the role has been created successfully, retrieve the ARN for the use in the steps to follow.

```bash
aws cloudformation describe-stacks --stack-name "cat-gen-sagemaker-role" \
    --query 'Stacks[0].Outputs[?OutputKey==`MLNotebookExecutionRole`].OutputValue' \
    --output text
```

It will look something like `arn:aws:iam::XXXXXXXXXXXX:role/cat-gen-sagemaker-role-ExecutionRole-PZL3SA3IZPSN`.

Next create a Code repository and pass it in our repo `https://github.com/t04glovern/aws-s3-cat-images`

``` bash
aws sagemaker create-code-repository \
    --code-repository-name "aws-s3-cat-images" \
    --git-config '{"Branch":"master", "RepositoryUrl" : "https://github.com/t04glovern/aws-s3-cat-images" }'
```

Finally create the notebook instance ensuring you pass in the Role ARN from before, and the default code repository we just created.

```bash
aws sagemaker create-notebook-instance \
    --notebook-instance-name "cat-gen" \
    --instance-type "ml.p2.xlarge" \
    --role-arn "arn:aws:iam::XXXXXXXXXXXXX:role/cat-gen-sagemaker-role-ExecutionRole-PZL3SA3IZPSN" \
    --default-code-repository "aws-s3-cat-images"
```

