# Lab 07: Terraform CI on GitHub Actions

## Overview

This lab teaches you how to implement Continuous Integration (CI) for Terraform using GitHub Actions. You'll learn to automatically run `terraform fmt`, `terraform validate`, and `terraform plan` on every push and pull request.

## Learning Objectives

- Set up GitHub Actions workflow for Terraform
- Automate Terraform formatting checks
- Validate Terraform configuration in CI
- Generate and review Terraform plans in pull requests
- Understand CI/CD best practices for Infrastructure as Code

## Prerequisites

- GitHub account
- Basic knowledge of Terraform
- Basic understanding of Git and GitHub
- Azure subscription (optional - for actual deployment)

## Repository Structure

```
07-terraform-ci-github-actions/
├── .github/
│   └── workflows/
│       └── terraform-ci.yml    # GitHub Actions workflow
├── .gitignore                  # Git ignore file
├── main.tf                     # Main Terraform configuration
├── providers.tf                # Provider configuration
├── variables.tf                # Variable definitions
├── outputs.tf                  # Output definitions
├── terraform.tfvars.example    # Example variables file
└── README.md                   # This file
```

## What This Lab Creates

This Terraform configuration creates a simple Azure Resource Group with tags. The focus is on the CI/CD pipeline, not the infrastructure complexity.

## Lab Steps

### Step 1: Create Repository from Template

1. Click "Use this template" → "Create a new repository"
2. Give your repository a name (e.g., `my-terraform-ci-lab`)
3. Choose public or private
4. Click "Create repository"

### Step 2: Clone Repository Locally

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPO-NAME.git
cd YOUR-REPO-NAME
```

### Step 3: (Optional) Test Terraform Locally

If you want to test the Terraform configuration locally:

```bash
# Initialize Terraform
terraform init

# Format the code
terraform fmt

# Validate the configuration
terraform validate

# Create a plan (requires Azure credentials)
terraform plan

# If you want to apply (optional - creates real resources)
terraform apply
```

**Note**: For `terraform plan` and `terraform apply`, you'll need to authenticate with Azure first:

```bash
az login
```

### Step 4: Make a Code Change

Create a feature branch and make a small change:

```bash
# Create a new feature branch
git checkout -b feature/add-new-tag

# Edit main.tf and add a new tag
# For example, add: Project = "CI-CD-Lab"
```

Edit `main.tf` and add a new tag to the resource group:

```hcl
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
    Lab         = "07-terraform-ci-github-actions"
    Project     = "CI-CD-Lab"  # Add this line
  }
}
```

### Step 5: Commit and Push Changes

```bash
git add main.tf
git commit -m "Add Project tag to resource group"
git push -u origin feature/add-new-tag
```

### Step 6: Create a Pull Request

1. Go to your repository on GitHub
2. Click "Compare & pull request"
3. Review the changes
4. Click "Create pull request"

### Step 7: Observe GitHub Actions Workflow

1. Navigate to the "Actions" tab in your repository
2. You should see a workflow running for your pull request
3. Click on the workflow to see the details

The workflow will:
- Check out your code
- Set up Terraform
- Run `terraform fmt -check` to verify formatting
- Run `terraform init` to initialize Terraform
- Run `terraform validate` to validate the configuration
- Run `terraform plan` to show what would change
- Comment on the PR with the plan output

### Step 8: Fix Formatting Issues (If Any)

If the `terraform fmt -check` step fails:

```bash
# Format the code
terraform fmt

# Commit the formatting changes
git add .
git commit -m "Fix Terraform formatting"
git push
```

The workflow will run again automatically.

### Step 9: Review the Plan in PR Comments

1. Go back to your pull request
2. Look for a comment from GitHub Actions with the Terraform plan
3. Review the plan to see what would change if you merged this PR

### Step 10: Merge the Pull Request

Once all checks pass:
1. Review the changes one more time
2. Click "Merge pull request"
3. Confirm the merge
4. Delete the feature branch (optional but recommended)

## Understanding the GitHub Actions Workflow

The workflow file `.github/workflows/terraform-ci.yml` does the following:

### Triggers

- **Push**: Runs on every push to any branch
- **Pull Request**: Runs when a PR is opened or updated targeting the `main` branch

### Jobs

The `terraform` job runs on `ubuntu-latest` and includes:

1. **Checkout**: Gets your code from the repository
2. **Setup Terraform**: Installs Terraform using the official HashiCorp action
3. **Format Check**: Ensures code is properly formatted (`terraform fmt -check`)
4. **Init**: Initializes Terraform and downloads providers
5. **Validate**: Validates the Terraform syntax and configuration
6. **Plan**: (PR only) Generates an execution plan
7. **Comment**: (PR only) Posts the plan output as a PR comment

## Key Concepts

### Terraform fmt

- Formats Terraform configuration files to a canonical format
- The `-check` flag makes it fail if files aren't formatted
- Helps maintain consistent code style across your team

### Terraform validate

- Validates the Terraform configuration syntax
- Checks for errors in your configuration
- Runs without accessing remote services

### Terraform plan

- Shows what Terraform will do before you apply changes
- Helps prevent unexpected changes
- In CI, this gives reviewers visibility into infrastructure changes

### CI/CD Best Practices

1. **Run on every push**: Catch issues early
2. **Block merges on failures**: Prevent broken code from entering main
3. **Review plans before apply**: Always know what will change
4. **Use local state for CI**: Simplifies the workflow (production would use remote state)

## Experiments to Try

1. **Intentionally break formatting**: Remove indentation and see the workflow fail
2. **Add invalid Terraform syntax**: See how `terraform validate` catches it
3. **Modify variables**: Change default values and observe the plan differences
4. **Add more resources**: Extend the configuration and see how CI handles it
5. **Add branch protection rules**: Require status checks to pass before merging

## Common Issues and Solutions

### Issue: Workflow doesn't run

- **Solution**: Check that the workflow file is in `.github/workflows/` directory
- Ensure the file has `.yml` or `.yaml` extension
- Check the Actions tab for any errors

### Issue: Terraform init fails

- **Solution**: Usually a provider version issue
- Check that `providers.tf` specifies valid provider versions
- Review the error message in the workflow logs

### Issue: Format check fails

- **Solution**: Run `terraform fmt` locally and commit the changes
- Make sure your editor doesn't auto-format in a different way

### Issue: Plan step fails

- **Solution**: For this lab, plan should work without credentials since it uses local backend
- If you've modified the code to use remote resources, you may need to add Azure credentials

## Next Steps

After completing this lab, consider:

1. **Add terraform apply**: Extend the workflow to apply changes (with approval)
2. **Use remote state**: Configure Azure Storage backend for state
3. **Add security scanning**: Use tools like tfsec or Checkov
4. **Implement environments**: Create separate workflows for dev/staging/prod
5. **Add cost estimation**: Integrate tools like Infracost to estimate costs

## Cleanup

If you deployed resources to Azure:

```bash
terraform destroy
```

This will remove the resource group and all associated resources.

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [HashiCorp Setup Terraform Action](https://github.com/hashicorp/setup-terraform)
- [Terraform CLI Documentation](https://www.terraform.io/docs/cli)
- [Azure Provider Documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)

## License

This lab is provided as-is for educational purposes.
