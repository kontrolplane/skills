---
name: terraform
description: Terraform infrastructure as code with HCL. Use when writing Terraform configurations, debugging state issues, understanding count vs for_each behavior, managing modules, or troubleshooting plan/apply errors.
---

# Terraform

## Common Gotchas

### count vs for_each
```hcl
# count: Index-based, problematic for changes
resource "aws_instance" "web" {
  count = 3  # Removing item 0 shifts all indexes, recreating 1 and 2
}

# for_each: Key-based, stable references
resource "aws_instance" "web" {
  for_each = toset(["a", "b", "c"])  # Removing "a" only affects "a"
}
```
**Use `for_each` unless you specifically need numeric indexing.**

### Dependency Cycles
Implicit dependencies via references usually sufficient:
```hcl
resource "aws_instance" "web" {
  subnet_id = aws_subnet.main.id  # Implicit dependency
}
```
Use `depends_on` only for hidden dependencies (IAM policies, etc.):
```hcl
depends_on = [aws_iam_role_policy_attachment.web]
```

### Sensitive Values
```hcl
output "password" {
  value     = random_password.db.result
  sensitive = true  # Hides in CLI output, NOT in state file
}
```
State file still contains plaintext. Always encrypt state at rest.

### lifecycle Rules
```hcl
lifecycle {
  create_before_destroy = true   # New resource before destroying old
  prevent_destroy       = true   # Block terraform destroy
  ignore_changes        = [tags] # Don't track this attribute
  replace_triggered_by  = [null_resource.trigger.id]  # Force replacement
}
```

## State Management

### Moving Resources
```bash
# Rename in state (no cloud change)
terraform state mv aws_instance.old aws_instance.new

# Move to module
terraform state mv aws_instance.web module.compute.aws_instance.web

# Remove from state only (keep cloud resource)
terraform state rm aws_instance.web
```

### Import
```bash
terraform import aws_instance.web i-1234567890abcdef0
```
After import, run `terraform plan` and adjust config until no changes.

### State Locking
DynamoDB for S3 backend prevents concurrent modifications:
```hcl
backend "s3" {
  bucket         = "state-bucket"
  key            = "terraform.tfstate"
  dynamodb_table = "terraform-locks"  # Required for locking
}
```

## Module Patterns

### Output Dependencies
Child module outputs implicitly depend on all module resources:
```hcl
# In module
output "instance_id" {
  value = aws_instance.web.id
}

# In root
resource "aws_eip" "web" {
  instance = module.compute.instance_id  # Waits for module completion
}
```

### Module Sources
```hcl
source = "./modules/vpc"                    # Local
source = "hashicorp/vpc/aws"                # Registry
source = "git::https://example.com/vpc.git" # Git
source = "git::https://example.com/vpc.git?ref=v1.0.0"  # Pinned
```

## Expressions

### Conditionals
```hcl
instance_type = var.env == "prod" ? "t3.large" : "t3.micro"
count         = var.create ? 1 : 0
```

### Splat and for
```hcl
instance_ids = aws_instance.web[*].id                    # Splat (count)
instance_ids = [for i in aws_instance.web : i.id]       # for (for_each)
instance_map = { for i in aws_instance.web : i.tags.Name => i.id }
```

### Null Coalescing
```hcl
value = var.custom_value != null ? var.custom_value : "default"
# Or with try():
value = try(var.config.nested.value, "default")
```
