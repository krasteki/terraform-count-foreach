# Learn Terraform count and for_each

Learn what Terraform count and for_each are and when to use them.

Follow along with these tutorials for
[count](https://learn.hashicorp.com/tutorials/terraform/count) and
[for_each](https://learn.hashicorp.com/tutorials/terraform/for-each) on
HashiCorp Learn.

I. Count

1. Refactor the EC2 configuration - Remove or comment out the entire block defining the `app_b` EC2 instance from `main.tf`

2. rename the resource for the other EC2 instance from `app_a` to `app`.

3. Declare a variable for instance number - add the `instances_per_subnet` variable to `variables.tf` to define how many instances each private subnet will have.

4. Scale EC2 configuration with `count` - edit `main.tf` to use `count` to provision multiple EC2 instances with the `app` resource block, based on the value of the new `instances_per_subnet` variable and the number of private subnets.

```
count = var.instances_per_subnet * length(module.vpc.private_subnets)
subnet_id = module.vpc.private_subnets[count.index % length(module.vpc.private_subnets)]
```

5. Update the load balancer - Update the load balancer configuration in the `elb_http` block to attach the instances to the load balancer.

```
number_of_instances = length(aws_instance.app)
instances           = aws_instance.app.*.id
```

6. Update `outputs.tf` to refer to the new `aws_instance.app` block instead of `app_a` and `app_b`

```
value       = aws_instance.app.*.id
```

7. Apply scalable configuration
```
$ terraform apply
```

II. For_each

1. create local module for the instances

2. Define a map to configure each project - Define a map for project configuration in `variables.tf` that `for_each` will iterate over to configure each resource.

3. Since the `project` variable includes most of the options that were configured by individual variables, comment out or remove these variables from `variables.tf`:
```
variable "project_name"
variable "environment"
variable "public_subnets_per_vpc"
variable "private_subnets_per_vpc"
variable "instance_type"
```

4. Add `for_each` to the VPC - use `for_each` to iterate over the `project` map in the `VPC` module block of `main.tf`, which will create one VPC for each key/value pair in the map.
Terraform will provision multiple VPCs, assigning each key/value pair in the `var.project` map to `each.key` and `each.value` respectively. With a list or set, `each.key` will be the index of the item in the collection, and `each.value` will be the value of the item.

5. Update the subnet configuration in the `vpc` module block in `main.tf` to use `each.value` to refer to these values.

6. Update the `app_security_group` module to iterate over the project variable to get the security group name, VPC ID, and CIDR blocks for each project.

7. Update the load balancer and its security group - Update the configuration for the load balancer security groups to iterate over the `project` variable to get their names and VPC IDs.

8. Update the `elb_http` block so that each VPC's load balancer name will also include the name of the `project`, the environment, and will use the corresponding security groups and subnets.

9. Move EC2 instance to a module - Remove the `resource "aws_instance" "app"` and `data "aws_ami" "amazon_linux"` blocks from your root module's `main.tf` file, and replace them with a reference to the `aws-instance` module.

10. replace the references to the EC2 instances in the `module "elb_http"` block with references to the new module.

11. replace the entire contents of 1outputs.tf1 in the root module with the following

```
output "public_dns_names" {
  description = "Public DNS names of the load balancers for each project"
  value       = { for p in sort(keys(var.project)) : p => module.elb_http[p].this_elb_dns_name }
}

output "vpc_arns" {
  description = "ARNs of the vpcs for each project"
  value       = { for p in sort(keys(var.project)) : p => module.vpc[p].vpc_arn }
}

output "instance_ids" {
  description = "IDs of EC2 instances"
  value       = { for p in sort(keys(var.project)) : p => module.ec2_instances[p].instance_ids }
}
```