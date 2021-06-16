Contents

1.	Note	2
2.	Scope of Automation	2
3.	Prerequisites	2
3.1	Valid Azure subscription	2
3.2	Install Azure CLI	3
3.3	Install Azure AKS CLI	3
3.4	Install Helm	3
3.5	Install Terraform	4
4.	Deploy AKS with Terraform Script	4
5.	Deploy Airflow with Terraform Script into a namespace	5



1.	Note
Steps captured in this version of documentation are verified on  Windows OS only. The same set up could be done on cloud shell as well. For Ubuntu or any other platform, this needs to tested.

2.	Scope of Automation
•	Launch AKS with a given Subscription & region.
•	Deploy Airflow (App version 2.1.0) from https://charts.bitnami.com/bitnami (chart version: 10.1.1)
3.	Prerequisites
•	Valid Azure subscription
•	Azure CLI (installed on the required workstation)
•	Azure AKS CLI  (installed on the required workstation)
•	Helm (installed on the required workstation)
•	Terraform (installed on the required workstation)

3.1	 Valid Azure subscription
Azure Subscription – At the least minimal free tier subscription should be active. (Based on the region number of CPU, RAM quotas will be applied. The details could be viewed by navigating to
Home -> subscriptions -> <subscription name>  -> settings(usage and Quotas)
Sample screenshot for reference:
 

3.2	 Install Azure CLI
For detailed installation steps across platforms refer to:
https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
Verify CLI login with command:
 az login
 

3.3	 Install Azure AKS CLI
AKS cli is needed to interact with the Kubernetes cluster. Use the below command to install AKS CLI.
az aks install-cli
Azure CLI will automatically download and configure the executables for kubectl and Azure kube login. On completion of set up ensure setting the environment PATH variables to include the path to the kubectl and azure kube login. It would be displayed on the terminal.
3.4	Install Helm
In Azure cloud shell this will be in built where on a local workstation we need to install this by following steps at https://helm.sh/docs/intro/install/
For windows:
•	Download the binary
•	Upzip in desired path
•	Add PATH to binary in system environment variables – PATH.
Restart the command terminal and run command to verify version: helm version 
 
3.5	Install Terraform
•	Download the binary https://www.terraform.io/downloads.html
•	unzip in desired path
•	Add PATH to binary in system environment variables – PATH.
 Restart the command line terminal and run the command to verify version: terraform -v
 

4.	Deploy AKS with Terraform Script

Create a service principal to allow terraform create resources into the subscription following the below steps using the Azure CLI and Save the output.
https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_secret
This command will output 5 values:
{
  "appId": "00000000-0000-0000-0000-000000000000",
  "displayName": "azure-cli-2017-06-05-10-41-15",
  "name": "http://azure-cli-2017-06-05-10-41-15",
  "password": "0000-0000-0000-0000-000000000000",
  "tenant": "00000000-0000-0000-0000-000000000000"
}
These values map to the Terraform variables like so:
•	appId is the client_id defined above.
•	password is the client_secret defined above.
•	tenant is the tenant_id defined above.

We need to set the client_id and client_secret as environment variables. In Unix like systems this could be done with export command where on windows we could use powershell cmdlet to achieve this. Below is the snippet of how the env vars were set:

 




Git clone the below Terraform script:


Navigate the folder containing the .tf files

Run the following commands:

•	terraform init

•	terraform plan (alternate way : terraform plan -out out.plan)

•	terraform apply  --auto-approve(alternate way if plan output is stored in out.plan: terraform apply out.plan)

Post apply the cluster specific auth params will be displayed, either save them in a config file or set as env vars of refresh kubeconfig login using: az aks get-credentials --resource-group <resource group name>  --name <cluster name>

5.	Deploy Airflow with Terraform Script into a namespace

Navigate to folder containing main.tf.

Ensure that the Kubernetes cluster config path is updated to the correct local kube config path. Refer below section of different mechanisms to capture the cluster login options

Run the following:

•	terraform init

•	terraform plan

•	terraform apply  --auto-approve

Finally we need to port forward the service to be accessible from the browser:
  
   kubectl port-forward svc/my-airflow-release 8080:8080 --namespace airflow2    
  
The deafult user without overiding values.yml is "user" and password could be obtained with the following command:
  
   kubectl get secret --namespace airflow2  my-airflow-release  -o jsonpath="{.data.airflow-password}" 










