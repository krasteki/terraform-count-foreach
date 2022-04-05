# Learn Terraform count and for_each

Learn what Terraform count and for_each are and when to use them.

Follow along with these tutorials for
[count](https://learn.hashicorp.com/tutorials/terraform/count) and
[for_each](https://learn.hashicorp.com/tutorials/terraform/for-each) on
HashiCorp Learn.


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