---
name: terraform-hashicorp
description: HashiCorp Terraform best practices — module creation, provider development, testing, Azure Verified Modules, Stacks, state import, and style guide. Auto-activates for Terraform/HCL projects.
origin: VoltAgent/awesome-agent-skills (hashicorp)
tools: Read, Write, Edit, Bash
---

# Terraform / HashiCorp Skills

Official HashiCorp skills for Terraform provider development, module authoring, and infrastructure management.

## Style Guide

```hcl
# terraform-style-guide: HashiCorp conventions

# 1. Use snake_case for all names
resource "aws_instance" "web_server" { }     # ✓
resource "aws_instance" "webServer" { }      # ✗

# 2. Group resource arguments: required → optional → blocks
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"   # required first
  instance_type = "t3.micro"

  tags = {                                   # optional after
    Name        = "web-server"
    Environment = var.environment
  }

  lifecycle {                                # blocks last
    create_before_destroy = true
  }
}

# 3. Use locals for repeated expressions
locals {
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

# 4. Prefer data sources over hardcoded values
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}
```

## Module Structure

```
modules/
  vpc/
    main.tf          # Resources
    variables.tf     # Input variables
    outputs.tf       # Output values
    versions.tf      # Required providers/versions
    README.md
  compute/
    main.tf
    variables.tf
    outputs.tf
    versions.tf

environments/
  prod/
    main.tf
    terraform.tfvars
  staging/
    main.tf
    terraform.tfvars
```

```hcl
# modules/vpc/variables.tf
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "Must be a valid CIDR block."
  }
}

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the created VPC"
  value       = aws_vpc.main.id
}
```

## Refactor to Modules

```bash
# terraform-search-import: discover existing resources
terraform plan -generate-config-out=generated.tf

# Import existing resources
terraform import aws_s3_bucket.my_bucket my-bucket-name
terraform import 'aws_instance.web["i-1234567890"]' i-1234567890

# Bulk import with import blocks (TF 1.5+)
```

```hcl
# Import block (Terraform 1.5+)
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}

# Generate config automatically
# terraform plan -generate-config-out=generated.tf
```

## Terraform Stacks (Multi-Environment)

```hcl
# stacks/main.tfstack.hcl
component "vpc" {
  source  = "./modules/vpc"
  version = "~> 2.0"

  inputs = {
    cidr        = var.vpc_cidr
    environment = var.environment
  }
}

component "compute" {
  source = "./modules/compute"

  inputs = {
    vpc_id      = component.vpc.vpc_id
    environment = var.environment
  }
}

# deployments.tfdeploy.hcl
deployment "production" {
  inputs = {
    environment = "prod"
    vpc_cidr    = "10.0.0.0/16"
  }
}

deployment "staging" {
  inputs = {
    environment = "staging"
    vpc_cidr    = "10.1.0.0/16"
  }
}
```

## Provider Development

```go
// Scaffold: hashicorp/new-terraform-provider
// Directory structure:
// internal/provider/
//   provider.go
//   resource_example.go
//   data_source_example.go

// Resource implementation
func (r *ExampleResource) Schema(_ context.Context, _ resource.SchemaRequest, resp *resource.SchemaResponse) {
    resp.Schema = schema.Schema{
        Attributes: map[string]schema.Attribute{
            "id": schema.StringAttribute{Computed: true},
            "name": schema.StringAttribute{Required: true},
            "tags": schema.MapAttribute{
                ElementType: types.StringType,
                Optional:    true,
            },
        },
    }
}

func (r *ExampleResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var data ExampleResourceModel
    resp.Diagnostics.Append(req.Plan.Get(ctx, &data)...)
    if resp.Diagnostics.HasError() { return }

    // Call API
    result, err := r.client.Create(data.Name.ValueString())
    if err != nil {
        resp.Diagnostics.AddError("Create Error", err.Error())
        return
    }

    data.ID = types.StringValue(result.ID)
    resp.Diagnostics.Append(resp.State.Set(ctx, &data)...)
}
```

## Acceptance Tests

```go
// hashicorp/provider-test-patterns
func TestAccExampleResource_basic(t *testing.T) {
    resource.Test(t, resource.TestCase{
        PreCheck:                 func() { testAccPreCheck(t) },
        ProtoV6ProviderFactories: testAccProtoV6ProviderFactories,
        Steps: []resource.TestStep{
            {
                Config: testAccExampleResourceConfig("test-name"),
                Check: resource.ComposeAggregateTestCheckFunc(
                    resource.TestCheckResourceAttr("example_resource.test", "name", "test-name"),
                    resource.TestCheckResourceAttrSet("example_resource.test", "id"),
                ),
            },
            // Import test
            {
                ResourceName:      "example_resource.test",
                ImportState:       true,
                ImportStateVerify: true,
            },
        },
    })
}

// Run acceptance tests
// TF_ACC=1 go test ./internal/provider/ -v -run TestAcc
```

## Testing Configurations

```hcl
# .tftest.hcl — built-in testing framework
run "basic_vpc_creation" {
  command = plan

  variables {
    vpc_cidr    = "10.0.0.0/16"
    environment = "test"
  }

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR should match input"
  }
}

run "vpc_has_required_tags" {
  command = apply

  assert {
    condition     = aws_vpc.main.tags["Environment"] == "test"
    error_message = "VPC must have Environment tag"
  }
}
```

```bash
# Run tests
terraform test
terraform test -filter=basic_vpc_creation
```

## Azure Verified Modules (AVM)

```hcl
# Use AVM-certified modules
module "storage_account" {
  source  = "Azure/avm-res-storage-storageaccount/azurerm"
  version = "~> 0.2"

  resource_group_name  = azurerm_resource_group.main.name
  location             = "eastus"
  name                 = "mystorageaccount"
  account_replication_type = "LRS"
  account_tier             = "Standard"

  # AVM interface: role_assignments, locks, tags
  role_assignments = {
    contributor = {
      role_definition_id_or_name = "Storage Blob Data Contributor"
      principal_id               = data.azurerm_client_config.current.object_id
    }
  }

  tags = local.common_tags
}
```

## Common Commands

```bash
terraform init          # Initialize
terraform fmt           # Format code
terraform validate      # Validate syntax
terraform plan          # Preview changes
terraform apply         # Apply changes
terraform destroy       # Destroy infrastructure
terraform state list    # List resources in state
terraform state show    # Inspect resource state
terraform output        # Show output values
terraform workspace new staging  # Create workspace
```

## Sources
- hashicorp/terraform-style-guide
- hashicorp/refactor-module
- hashicorp/new-terraform-provider
- hashicorp/provider-resources
- hashicorp/provider-test-patterns
- hashicorp/run-acceptance-tests
- hashicorp/terraform-test
- hashicorp/terraform-search-import
- hashicorp/terraform-stacks
- hashicorp/azure-verified-modules
