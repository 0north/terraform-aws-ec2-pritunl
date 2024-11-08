# AWS EC2 Pritunl Module

This Terraform module is designed to deploy a Pritunl VPN server on AWS. It
automates the creation of necessary resources including an EC2 instance with
Ubuntu, Elastic IP, required IAM roles and policies for access management, an S3
bucket for backups, CloudWatch Logs for logging, and configures security through
Security Groups.

#### Features

- **EC2 Instance**: Automatically creates an EC2 instance using a specified
  Ubuntu AMI.
- **IAM Roles and Policies**: Creates IAM roles and policies for managing access
  to AWS resources.
- **S3 Bucket**: Optionally creates an S3 bucket for storing backups.
- **CloudWatch Logs**: Optionally configures CloudWatch Logs for server logging.
- **Security Groups**: Configures inbound and outbound rules via Security
  Groups.
- **Elastic IP**: Associates an Elastic IP with the EC2 instance.
- **SSH Key Pair**: Optionally creates an SSH key for access to the EC2
  instance.
- **Route53 Record**: Optionally creates a DNS record in Route53 for the EC2
  instance.
- **SSM Parameters**: Stores configuration data in SSM Parameter Store.

#### Usage

To use this module, add the following code to your `main.tf` file, replacing
parameters with your own values:

```hcl
module "pritunl" {
  source = "iops-team/ec2-pritunl/aws"

  name                       = "example-pritunl"
  create_ssh_key             = true
  backups                    = true
  backups_cron               = "cron(0 0 * * ? *)"
  instance_type              = "t3.micro"
  vpc_id                     = "vpc-0d7be8904638ef5fb"
  subnet_id                  = "subnet-06442a69eb3006b2e"
  monitoring                 = true
  cloudwatch_logs            = true
  wait_for_installation      = true
  create_route53_record      = true
  zone_id                    = "Z069875012GQNG6IN9LUI"
  domain_name                = "vpn.example.com"

  tags = {
    Environment = "production"
    Project     = "Pritunl VPN"
  }

  ingress_rules = [
    {
      from_port   = 80,
      to_port     = 80,
      protocol    = "tcp",
      cidr_blocks = ["0.0.0.0/0"],
      description = "HTTP access"
    },
    {
      from_port   = 443,
      to_port     = 443,
      protocol    = "tcp",
      cidr_blocks = ["0.0.0.0/0"],
      description = "HTTPS access"
    },
    {
      from_port   = 1194,
      to_port     = 1194,
      protocol    = "udp",
      cidr_blocks = ["0.0.0.0/0"],
      description = "VPN access"
    }
  ]

  egress_rules = [
    {
      from_port   = 0,
      to_port     = 0,
      protocol    = "-1",
      cidr_blocks = ["0.0.0.0/0"],
      description = "Allow all outbound traffic"
    }
  ]

  root_block_device = [
    {
      volume_type           = "gp3",
      throughput            = 125,
      volume_size           = 20,
      encrypted             = true,
      kms_key_id            = "",
      iops                  = 3000,
      delete_on_termination = true
    }
  ]

  s3_lifecycle_rule = [
    {
      id                                     = "expireBackups",
      enabled                                = true,
      abort_incomplete_multipart_upload_days = 7,
      expiration = {
        days                         = 30,
        expired_object_delete_marker = false
      },
      noncurrent_version_expiration = [
        {
          noncurrent_days = 60
        }
      ]
    }
  ]
}
```

## Requirements

| Name                                                                     | Version  |
| ------------------------------------------------------------------------ | -------- |
| <a name="requirement_terraform"></a> [terraform](#requirement_terraform) | >= 0.12  |
| <a name="requirement_aws"></a> [aws](#requirement_aws)                   | ~> 5.0   |
| <a name="requirement_null"></a> [null](#requirement_null)                | ~> 3.1.0 |

## Providers

| Name                                                | Version  |
| --------------------------------------------------- | -------- |
| <a name="provider_aws"></a> [aws](#provider_aws)    | ~> 5.0   |
| <a name="provider_null"></a> [null](#provider_null) | ~> 3.1.0 |

## Modules

| Name                                                        | Source                              | Version |
| ----------------------------------------------------------- | ----------------------------------- | ------- |
| <a name="module_key_pair"></a> [key_pair](#module_key_pair) | terraform-aws-modules/key-pair/aws  | 2.0.0   |
| <a name="module_s3"></a> [s3](#module_s3)                   | terraform-aws-modules/s3-bucket/aws | ~> 3.0  |

## Resources

| Name                                                                                                                                                                     | Type        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- |
| [aws_cloudwatch_log_group.pritunl_log_group](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_log_group)                           | resource    |
| [aws_eip.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip)                                                                       | resource    |
| [aws_eip_association.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/eip_association)                                               | resource    |
| [aws_iam_instance_profile.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_instance_profile)                                     | resource    |
| [aws_iam_policy.additional_instance_role_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)                                 | resource    |
| [aws_iam_policy.backups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)                                                         | resource    |
| [aws_iam_policy.cloudwatch_logs_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)                                          | resource    |
| [aws_iam_policy.ssm_put_parameter](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)                                               | resource    |
| [aws_iam_policy.ssm_send_command_pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy)                                        | resource    |
| [aws_iam_policy_attachment.ssm_put_parameter_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy_attachment)              | resource    |
| [aws_iam_policy_attachment.ssm_send_command_attachment](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_policy_attachment)               | resource    |
| [aws_iam_role.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role)                                                             | resource    |
| [aws_iam_role_policy_attachment.additional_instance_role_policy](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment) | resource    |
| [aws_iam_role_policy_attachment.backups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment)                         | resource    |
| [aws_iam_role_policy_attachment.cloudwatch_logs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment)                 | resource    |
| [aws_iam_role_policy_attachment.ssm](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_role_policy_attachment)                             | resource    |
| [aws_instance.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)                                                             | resource    |
| [aws_route53_record.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route53_record)                                                 | resource    |
| [aws_security_group.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group)                                                 | resource    |
| [aws_ssm_association.backup](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_association)                                                | resource    |
| [aws_ssm_document.backups_sript](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_document)                                               | resource    |
| [aws_ssm_document.cloud_init_wait](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_document)                                             | resource    |
| [aws_ssm_document.restore_mongodb](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_document)                                             | resource    |
| [aws_ssm_parameter.default_credential](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_parameter)                                        | resource    |
| [aws_ssm_parameter.ec2_keypair](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_parameter)                                               | resource    |
| [null_resource.wait_for_installation_pritunl](https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource)                                     | resource    |
| [aws_ami.ubuntu](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ami)                                                                     | data source |
| [aws_availability_zones.available](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/availability_zones)                                    | data source |
| [aws_caller_identity.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/caller_identity)                                            | data source |
| [aws_eip.pritunl](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/eip)                                                                    | data source |
| [aws_iam_policy_document.backups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/iam_policy_document)                                    | data source |
| [aws_region.current](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/region)                                                              | data source |

## Inputs

| Name                                                                                                                                          | Description                                                                                                                 | Type                                                                                                                                                                                                                                                                                                      | Default                                                                                                                                                                                                                                                                                                                               | Required |
| --------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------: |
| <a name="input_additional_instance_role_policy_json"></a> [additional_instance_role_policy_json](#input_additional_instance_role_policy_json) | Additional JSON formatted IAM policy to attach to the Pritunl EC2 instance role.                                            | `string`                                                                                                                                                                                                                                                                                                  | `null`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_additional_user_data"></a> [additional_user_data](#input_additional_user_data)                                                 | Additional user data script to execute after Pritunl has started.                                                           | `string`                                                                                                                                                                                                                                                                                                  | `""`                                                                                                                                                                                                                                                                                                                                  |    no    |
| <a name="input_auto_restore"></a> [auto_restore](#input_auto_restore)                                                                         | Try to restore from the backup if it exists, if the backup does not exist, a new user will be created                       | `bool`                                                                                                                                                                                                                                                                                                    | `true`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_backups"></a> [backups](#input_backups)                                                                                        | Flag to determine if an S3 bucket should be created.                                                                        | `bool`                                                                                                                                                                                                                                                                                                    | n/a                                                                                                                                                                                                                                                                                                                                   |   yes    |
| <a name="input_backups_cron"></a> [backups_cron](#input_backups_cron)                                                                         | Cron schedule for MongoDB backups                                                                                           | `string`                                                                                                                                                                                                                                                                                                  | `"cron(0 0 * * ? *)"`                                                                                                                                                                                                                                                                                                                 |    no    |
| <a name="input_cloudwatch_logs"></a> [cloudwatch_logs](#input_cloudwatch_logs)                                                                | A flag to enable or disable log streaming to CloudWatch                                                                     | `bool`                                                                                                                                                                                                                                                                                                    | `false`                                                                                                                                                                                                                                                                                                                               |    no    |
| <a name="input_cloudwatch_logs_group_name"></a> [cloudwatch_logs_group_name](#input_cloudwatch_logs_group_name)                               | The name of the CloudWatch Logs Group for Pritunl logs                                                                      | `string`                                                                                                                                                                                                                                                                                                  | `null`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_cloudwatch_logs_retention_in_days"></a> [cloudwatch_logs_retention_in_days](#input_cloudwatch_logs_retention_in_days)          | Retention in days to configure for the CloudWatch log group                                                                 | `number`                                                                                                                                                                                                                                                                                                  | `30`                                                                                                                                                                                                                                                                                                                                  |    no    |
| <a name="input_create_route53_record"></a> [create_route53_record](#input_create_route53_record)                                              | Whether to create Route53 record                                                                                            | `bool`                                                                                                                                                                                                                                                                                                    | `false`                                                                                                                                                                                                                                                                                                                               |    no    |
| <a name="input_create_ssh_key"></a> [create_ssh_key](#input_create_ssh_key)                                                                   | Flag to determine if an SSH key should be created.                                                                          | `bool`                                                                                                                                                                                                                                                                                                    | `true`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_domain_name"></a> [domain_name](#input_domain_name)                                                                            | The domain name for the Route53 record                                                                                      | `string`                                                                                                                                                                                                                                                                                                  | `null`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_egress_rules"></a> [egress_rules](#input_egress_rules)                                                                         | Default egress rules for the security group                                                                                 | <pre>list(object({<br> from_port = number<br> to_port = number<br> protocol = string<br> cidr_blocks = list(string)<br> description = string<br> }))</pre>                                                                                                                                                | <pre>[<br> {<br> "cidr_blocks": [<br> "0.0.0.0/0"<br> ],<br> "description": "Allow all outbound traffic",<br> "from_port": 0,<br> "protocol": "-1",<br> "to_port": 0<br> }<br>]</pre>                                                                                                                                                 |    no    |
| <a name="input_eip_id"></a> [eip_id](#input_eip_id)                                                                                           | The allocation ID of an existing Elastic IP to associate with the Pritunl instance. If unset, will create a new Elastic IP. | `string`                                                                                                                                                                                                                                                                                                  | `null`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_ingress_rules"></a> [ingress_rules](#input_ingress_rules)                                                                      | Default ingress rules for the security group                                                                                | <pre>list(object({<br> from_port = number<br> to_port = number<br> protocol = string<br> cidr_blocks = list(string)<br> description = string<br> }))</pre>                                                                                                                                                | <pre>[<br> {<br> "cidr_blocks": [<br> "0.0.0.0/0"<br> ],<br> "description": "HTTP access",<br> "from_port": 80,<br> "protocol": "tcp",<br> "to_port": 80<br> },<br> {<br> "cidr_blocks": [<br> "0.0.0.0/0"<br> ],<br> "description": "HTTPS access",<br> "from_port": 443,<br> "protocol": "tcp",<br> "to_port": 443<br> }<br>]</pre> |    no    |
| <a name="input_instance_type"></a> [instance_type](#input_instance_type)                                                                      | The instance type of the EC2 instance.                                                                                      | `string`                                                                                                                                                                                                                                                                                                  | `"t3.micro"`                                                                                                                                                                                                                                                                                                                          |    no    |
| <a name="input_monitoring"></a> [monitoring](#input_monitoring)                                                                               | Whether to enable detailed monitoring for the Pritunl instance                                                              | `bool`                                                                                                                                                                                                                                                                                                    | `false`                                                                                                                                                                                                                                                                                                                               |    no    |
| <a name="input_name"></a> [name](#input_name)                                                                                                 | The name of the resources.                                                                                                  | `string`                                                                                                                                                                                                                                                                                                  | n/a                                                                                                                                                                                                                                                                                                                                   |   yes    |
| <a name="input_root_block_device"></a> [root_block_device](#input_root_block_device)                                                          | Configuration for the root block device                                                                                     | <pre>list(object({<br> volume_type = string<br> throughput = number<br> volume_size = number<br> encrypted = bool<br> kms_key_id = string<br> iops = number<br> delete_on_termination = bool<br> }))</pre>                                                                                                | <pre>[<br> {<br> "delete_on_termination": true,<br> "encrypted": true,<br> "iops": 0,<br> "kms_key_id": "",<br> "throughput": 200,<br> "volume_size": 30,<br> "volume_type": "gp3"<br> }<br>]</pre>                                                                                                                                   |    no    |
| <a name="input_s3_force_destroy"></a> [s3_force_destroy](#input_s3_force_destroy)                                                             | Enables Terraform to forcibly destroy the bucket with backups, permanently deleting its contents                            | `bool`                                                                                                                                                                                                                                                                                                    | `false`                                                                                                                                                                                                                                                                                                                               |    no    |
| <a name="input_s3_lifecycle_rule"></a> [s3_lifecycle_rule](#input_s3_lifecycle_rule)                                                          | Lifecycle rule for the S3 bucket                                                                                            | <pre>list(object({<br> id = string<br> enabled = bool<br> abort_incomplete_multipart_upload_days = number<br> expiration = object({<br> days = number<br> expired_object_delete_marker = bool<br> })<br> noncurrent_version_expiration = list(object({<br> noncurrent_days = number<br> }))<br> }))</pre> | <pre>[<br> {<br> "abort_incomplete_multipart_upload_days": 1,<br> "enabled": true,<br> "expiration": {<br> "days": 0,<br> "expired_object_delete_marker": true<br> },<br> "id": "removeOldVersions",<br> "noncurrent_version_expiration": [<br> {<br> "noncurrent_days": 14<br> }<br> ]<br> }<br>]</pre>                              |    no    |
| <a name="input_subnet_id"></a> [subnet_id](#input_subnet_id)                                                                                  | The subnet ID where the EC2 instance will be launched.                                                                      | `string`                                                                                                                                                                                                                                                                                                  | n/a                                                                                                                                                                                                                                                                                                                                   |   yes    |
| <a name="input_tags"></a> [tags](#input_tags)                                                                                                 | A map of tags to add to all resources.                                                                                      | `map(string)`                                                                                                                                                                                                                                                                                             | `null`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_vpc_id"></a> [vpc_id](#input_vpc_id)                                                                                           | The VPC ID where the security group and EC2 instance will be created.                                                       | `string`                                                                                                                                                                                                                                                                                                  | n/a                                                                                                                                                                                                                                                                                                                                   |   yes    |
| <a name="input_wait_for_installation"></a> [wait_for_installation](#input_wait_for_installation)                                              | Whether to wait for the completion of the init.sh script on the EC2 instance                                                | `bool`                                                                                                                                                                                                                                                                                                    | `true`                                                                                                                                                                                                                                                                                                                                |    no    |
| <a name="input_zone_id"></a> [zone_id](#input_zone_id)                                                                                        | The Route53 zone ID where the record will be created                                                                        | `string`                                                                                                                                                                                                                                                                                                  | `null`                                                                                                                                                                                                                                                                                                                                |    no    |

## Outputs

| Name                                                                                                                                | Description                                                                           |
| ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| <a name="output_cloudwatch_log_group_name"></a> [cloudwatch_log_group_name](#output_cloudwatch_log_group_name)                      | The name of the CloudWatch Logs group for Pritunl logs.                               |
| <a name="output_eip_public_ip"></a> [eip_public_ip](#output_eip_public_ip)                                                          | The Elastic IP address associated with the Pritunl instance.                          |
| <a name="output_iam_instance_profile_name"></a> [iam_instance_profile_name](#output_iam_instance_profile_name)                      | The name of the IAM instance profile associated with the Pritunl instance.            |
| <a name="output_iam_role_name"></a> [iam_role_name](#output_iam_role_name)                                                          | The name of the IAM role created for Pritunl.                                         |
| <a name="output_instance_id"></a> [instance_id](#output_instance_id)                                                                | The ID of the Pritunl EC2 instance.                                                   |
| <a name="output_route53_fqdn"></a> [route53_fqdn](#output_route53_fqdn)                                                             | The DNS name for the Pritunl instance.                                                |
| <a name="output_s3_bucket_arn"></a> [s3_bucket_arn](#output_s3_bucket_arn)                                                          | The ARN of the S3 bucket used for backups.                                            |
| <a name="output_s3_bucket_id"></a> [s3_bucket_id](#output_s3_bucket_id)                                                             | The ID of the S3 bucket used for backups.                                             |
| <a name="output_security_group_id"></a> [security_group_id](#output_security_group_id)                                              | The ID of the security group created for Pritunl.                                     |
| <a name="output_ssm_parameter_default_credential"></a> [ssm_parameter_default_credential](#output_ssm_parameter_default_credential) | The name of the SSM parameter storing the default credentials of the Pritunl service. |
| <a name="output_ssm_parameter_key_pair"></a> [ssm_parameter_key_pair](#output_ssm_parameter_key_pair)                               | The name of the SSM parameter storing the SSH private key of the Pritunl instance.    |

### Notes on Possible Values

- **Instance Type**: The instance type can be any that AWS supports. Common
  types include `t2.micro`, `t3.medium`, `m5.large`, but this is subject to
  AWS's current offerings.
- **Boolean Flags**: For `bool` type variables, `true` enables and `false`
  disables the feature or resource creation.
- **IDs and Names**: Values for IDs (`vpc_id`, `subnet_id`, `zone_id`) and names
  (`name`, `domain_name`, `cloudwatch_logs_group_name`) depend on the resources
  you have available in your AWS account and the naming conventions you wish to
  follow.
- **Cron Schedule**: The `backups_cron` follows the standard cron format. You
  can customize this schedule to your backup frequency requirements.

This table aims to provide clarity on the input variables, ensuring users can
tailor the module to fit their specific AWS infrastructure and security needs.

## License

This module is released under the
[MIT License](https://opensource.org/licenses/MIT).

