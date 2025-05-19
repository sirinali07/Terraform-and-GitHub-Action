## Using Output Feature 

### Task-1: Using output feature of Terraform to get the IP Address of EC2 Instance
```
mkdir output-variable-lab && cd output-variable-lab
```
```
ls
```
Create the instance.tf file
```
vi instance.tf
```
```
provider "aws" {
  region     = "us-east-1"
}

resource "aws_instance" "example" {
  ami           = "ami-084568db4383264d4"
  instance_type = "t2.micro"
}
```
Save the changes
```
vi output.tf
```
```
output "Public_ip" {
  description = "Public IP of the instance"
  value = aws_instance.example.public_ip
}

output "Private_ip" {
  sensitive = true
  description = "Private IP of the instance"
  value = aws_instance.example.private_ip

}
```
Save the file



Once replaced, save the changes.
```
terraform init
```
```
terraform plan
```
```
terraform apply
```
```
terraform output Public_ip
```
```
terraform output Private_ip
```
Use the `terraform destroy` command to clean the infrastructure used in this lab.
```
terraform destroy
```
Once done, remove the directory and Zip file using the `rm -rf` command below.
```
cd ~
rm -rf output-variable-lab
```


