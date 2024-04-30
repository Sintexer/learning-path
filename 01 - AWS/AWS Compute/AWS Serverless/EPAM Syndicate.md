EPAM Syndicate is a tool similar to [[AWS SAM]] that helps building [[AWS Lambda]] application configurations. It allows you to easily deploy serverless applications using resource descriptions


## How Syndicate works

First one must init a project. Project is a directory where syndicate stores configs. Project contains build tool related files. For java its `pom.xml` and `jsrc` folder with sources. Syndicate stores configs in `.syndicate-config-${name}` where **name** is a config name.

`.syndicate-config-${name}` folder contains **syndicate.yml**, **syndicate_aliases.yml**, **bundles/**, and **logs/**. Bundles are stored in **bundles/** where each build generates separate bundle with name including date and time (e.g. `task02_240428.163634`). **syndicate.yml** contains account information like account id, target bucket for deploying bundles, temp credentials, resources prefix, region, project path and some other project settings. 

> [!warning]
> Whole `.syndicate-config-${name}` folder must be excluded from version control since it contains credentials and sensitive information.

Another important syndicate file is `deployment_resources.json`. This file specifies resources vital for lambda deploy, such as lambda role and associated policies. An example can be found below:

```json
{
  "lambda-basic-execution": {
    "policy_content": {
      "Statement": [
        {
          "Action": [
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents",
            "dynamodb:GetItem",
            "dynamodb:Query",
            "dynamodb:PutItem",
            "dynamodb:Batch*",
            "dynamodb:DeleteItem",
            "ssm:PutParameter",
            "ssm:GetParameter",
            "kms:Decrypt"
          ],
          "Effect": "Allow",
          "Resource": "*"
        }
      ],
      "Version": "2012-10-17"
    },
    "resource_type": "iam_policy"
  },
  "hello_world-role": {
    "predefined_policies": [],
    "principal_service": "lambda",
    "custom_policies": [
      "lambda-basic-execution"
    ],
    "resource_type": "iam_role"
  }
}

```

### 2.2 Usage guide

#### 2.2.1 Creating Project files

Execute `syndicate generate project` command to generate the project with all the necessary components and in a right folders/files hierarchy to start developing in a min. Command example:

```shell
syndicate generate project 
    --name $project_name
    --config_path $path_to_project
```

All the provided information is validated. After the project folder will be generated the command will return the following message:

```shell
Project name: $project_name
Project path: $path_to_project
```

The following files will be created in this folder: .gitignore, .syndicate, CHANGELOG.md, deployment_resources.json, README.md.

Command sample:

```shell
syndicate generate project --name DemoSyndicateJava && cd DemoSyndicateJava
```

For more details please execute `syndicate generate project --help`

#### 2.2.2 Creating configuration files for environment

Execute the `syndicate generate config` command to create Syndicate configuration files. Command example:

```shell
syndicate generate config
    --name                      $configuration_name   [required]
    --region                    $region_name          [required]
    --bundle_bucket_name        $s3_bucket_name       [required]
    --access_key                $access_key  
    --secret_key                $secret_key   
    --config_path               $path_to_store_config
    --project_path              $relative_path_to_project
    --prefix                    $prefix
    --suffix                    $suffix
    --extended_prefix           $extended_prefix_mode
    --use_temp_creds            $use_temp_creds #Specify,if use mfa or access_role
    --access_role               $role_name
    --serial_number             $serial_number
    --tags                      $KEY:VALUE
    --iam_permissions_boundary  $ARN 
```

All the provided information is validated.

> _Note:_ you may not specify `--access_key` and `--secret_key` params. In this case Syndicate will try to find your credentials by the path `~/.aws` or from environment variables.

> _Note:_ You can force Syndicate to generate temporary credentials and use them for deployment. For such cases, set `use_temp_creds` parameter to `True` and specify serial number if IAM user which will be used for deployment has a policy that requires MFA authentication. Syndicate will prompt you to enter MFA code to generate new credentials, save and use them until expiration.

After the configuration files will be generated the command will return the following message:

```shell
 Syndicate initialization has been completed. 
 Set SDCT_CONF:
 Unix: export SDCT_CONF=$path_to_store_config
 Windows: setx SDCT_CONF $path_to_store_config
```

Just copy one of the last two lines, depending on your OS, and execute the command. The commands set the environment variable SDCT_CONF required by aws-syndicate to operate.

> _Note:_ The default syndicate_aliases.yaml file has been generated. Your application may require additional aliases to be deployed - please add them to the file.

Command sample:

```shell
SYNDICATE_AWS_ACCESS_KEY=# enter your aws_access_key_id here
SYNDICATE_AWS_SECRET_KEY=# enter your aws_secret_access_key here
syndicate generate config --name dev --region eu-central-1 --bundle_bucket_name syndicate-artifacts-eu-central-1 --access_key $SYNDICATE_AWS_ACCESS_KEY --secret_key $SYNDICATE_AWS_SECRET_KEY --config_path $(pwd) --prefix syn- --suffix -dev --tags ENV:DEV
```

For more details please execute `syndicate generate config --help`

#### 2.2.3 Creating lambda files

Execute `syndicate generate lambda` command to generate required environment for lambda function except business logic. Command example:

```shell
syndicate generate lambda
    --name $lambda_name_1
    --runtime python|java|nodejs
    --project_path $project_path
```

All the provided information is validated. Different environments will be created for different runtimes:

- for Python

```
    .
    ├── $project_path
    │   └── src
    │       ├── commons
    │       │   ├── __init__.py
    │       │   ├── abstract_lambda.py
    │       │   ├── exception.py
    │       │   └── log_helper.py
    │       └── lambdas
    │           ├── $lambda_name_1
    │           │   ├── __init__.py
    │           │   ├── deployment_resources.json
    │           │   ├── handler.py
    │           │   ├── lambda_config.json
    │           │   ├── local_requirements.txt
    │           │   └── requirements.txt
    │           ├── $lambda_name_2
    │           │   ├── __init__.py
    │           │   ├── deployment_resources.json
    │           │   ├── handler.py
    │           │   ├── lambda_config.json
    │           │   ├── local_requirements.txt
    │           │   └── requirements.txt
    │           ├── __init__.py
    │           └── ...
    └── ...
```

- for Java

```
    .
    ├── $project_path
    │   └── jsrc
    │       └── main
    │           └── java
    │               └── com
    │                   └── $projectpath
    │                       ├── $lambda_name_1.java
    │                       └── $lambda_name_2.java
    └── ...
```

- for NodeJS

```
    .
    ├── $project_path
    │   └── app
    │       └── lambdas
    │           ├── $lambda_name_1
    │           │   ├── deployment_resources.json
    │           │   ├── lambda_config.json
    │           │   ├── index.js
    │           │   ├── package.json
    │           │   └── package-lock.json
    │           └── $lambda_name_2
    │               ├── deployment_resources.json
    │               ├── lambda_config.json
    │               ├── index.js
    │               ├── package.json
    │               └── package-lock.json
    └── ...
```

Command sample:

```shell
syndicate generate lambda --name DemoLambda --runtime java --project_path $(pwd)
```

For more details please execute `syndicate generate lambda --help`

### 2.3 Deployment

#### 2.3.1 Create an S3 bucket for aws-syndicate artifacts:

```shell
syndicate create_deploy_target_bucket
```

#### 2.3.2 Build project artifacts

```shell
syndicate build
```

#### 2.3.3 Deploy project resources to AWS account

```shell
syndicate deploy
```

Now the DemoLambda is created and available to be tested.

#### 2.3.4 Update resources

In order to be sure your latest changes works well on the AWS account the application should be deployed to the AWS account. To do this use the following commands set:

```shell
syndicate build
syndicate update
```

#### 2.3.5 Lambdas invocations metrics

```shell
syndicate profiler
```

#### 2.3.6 Clean up project resources from AWS Account

syndicate clean 

#### 2.3.7 Observing the environment manipulation history

```shell
syndicate status # this shows the general CLI dashboard where latest modification, locks state, latest event, project resources are shown
syndicate status --events # this returns all the history of what happened to the environment
```

### 2.4 Export

The command is aimed at exporting resource configurations

Parameters:

resource_type: The type of resource to export configuration  
dsl: Output domain-specific language

#### 2.4.1 API Gateway OpenAPI specification

```
syndicate export
 --resource_type api_gateway
 --dsl oas_v3
```

By default, the specification will be saved in the export directory inside the project root directory as `<api_id>_oas_v3.json`.

## Documentation

[Syndicate project examples](https://github.com/epam/aws-syndicate/tree/master/examples)

You can find a detailed documentation [here](https://github.com/epam/aws-syndicate/wiki)

## Developing lambdas

**Lambda function URL config**. To enable URL config, add @LambdaUrlConfig annotation and remove aliasName from @LambdaHandler.