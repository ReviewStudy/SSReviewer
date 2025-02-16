# Terraformer: Importing Infrastructure as Code
#Terraformer #InfrastructureAsCode #Terraform #AWS #Import

**Summary**: Terraformer is an open-source tool that converts existing infrastructure into Terraform configuration files, allowing users to manage their infrastructure as code.

## Content

Terraformer is an open-source tool designed to convert existing infrastructure into Terraform configuration files. By using commands like `terraformer import`, users can import their current infrastructure setup into Terraform. For example, the command:

```bash
terraformer import aws --resources=* --path-pattern="{output}/" --connect=true --regions=ap-northeast-2 --profile ""
```

enables users to import all resource files into Terraform.

### How Does It Work?
- Terraformer utilizes the provider's refresh method to gather all data.
- This data is then converted into Go structs.
- It generates HCL and JSON files.
- A `tfstate` file is created to reflect the current state of the infrastructure.

### What is Terraform Refresh?
- The Terraform state file is synchronized with the current state of the infrastructure managed by Terraform.
- This is necessary because manual changes might be made to the infrastructure, which need to be reflected in the Terraform state file.
- The primary purpose of Terraform Refresh is to detect any drift between the actual state of resources and the desired state as defined in the configuration files.

> The main goal of Terraform Refresh is to identify any discrepancies between the actual state of resources and the desired state as defined in the configuration files.

This process does not modify the actual remote objects but updates the Terraform state file. Terraformer uses this to generate Terraform files. Although Terraform automatically performs this during `terraform plan` and `terraform apply`, it is not always necessary.

```go
func Import(provider terraformutils.ProviderGenerator, options ImportOptions, args []string) error {

	// Initialize provider object and options, create providerWrapper
	providerWrapper, options, err := initOptionsAndWrapper(provider, options, args)
	if err != nil {
		return err
	}
	defer providerWrapper.Kill() // Clean up providerWrapper on function exit

	// Create ProviderMapping object based on provider
	providerMapping := terraformutils.NewProvidersMapping(provider)

	// Initialize resources for all services from provider
	err = initAllServicesResources(providerMapping, options, args, providerWrapper)
	if err != nil {
		return err
	}

	// Refresh current resource state from provider
	err = terraformutils.RefreshResourcesByProvider(providerMapping, providerWrapper)
	if err != nil {
		return err
	}

	// Convert Terraform state file and reflect in providerMapping
	providerMapping.ConvertTFStates(providerWrapper)

	// Organize structs with additional data per resource
	providerMapping.CleanupProviders()

	// Execute resource import based on import plan
	err = importFromPlan(providerMapping, options, args)

	return err // Return any final error
}
```

```bash
sudo terraformer import aws --resources="ecs" --path-pattern="{output}/" --regions=ap-northeast-2 --profile ""
```

Executing the above command imports the current resources, as shown in the image below:

![Pasted image 20250217001044.png]

## Key Points
- Terraformer converts existing infrastructure into Terraform configuration files.
- It uses the provider's refresh method to gather data and create HCL, JSON, and `tfstate` files.
- Terraform Refresh synchronizes the Terraform state file with the current infrastructure state.
- The process identifies discrepancies between actual and desired resource states.

## Follow-up Questions / Discussion Points
1. How does Terraformer handle changes made directly to the infrastructure outside of Terraform?
2. What are the benefits of using Terraformer for infrastructure management?
3. Can Terraformer be used with cloud providers other than AWS?
4. How does Terraform Refresh contribute to maintaining infrastructure consistency?
5. What are the potential challenges when using Terraformer to import large-scale infrastructures?