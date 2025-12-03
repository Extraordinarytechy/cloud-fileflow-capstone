## **Project Overview**

This DevOps Accelerator demonstrates a complete cloud-native flow where users can:

* Upload input files through a frontend hosted on S3 + CloudFront
* Use secure pre-signed URLs for file uploads
* Trigger backend processing through S3 events and AWS Lambda
* Deploy and manage infrastructure using Terraform
* Automate deployments using GitHub Actions
* Monitor logs and health using CloudWatch
* Receive notifications via SNS for processed files

---

## **Workflow (How It Works)**

1. The user accesses the frontend application through the browser.
2. Navigates through all available sections.
3. Completes the payment and uploads a screenshot or PDF file.
4. The uploaded file is converted into a pre-signed URL and stored in the S3 bucket.
5. An S3 event triggers the Lambda function.
6. The Lambda function processes the file and logs the results in CloudWatch.
7. An SNS alert is sent to the owner once processing is completed successfully.

---

## **Tech Stack**

| Layer           | Tools & Services                              |
| --------------- | --------------------------------------------- |
| Frontend        | HTML/CSS, S3, CloudFront                      |
| Backend (Event) | AWS Lambda (Python)                           |
| Infrastructure  | Terraform (modular setup with remote backend) |
| CI/CD           | GitHub Actions (workflow automation)          |
| Monitoring      | CloudWatch (logs, alarms, dashboards)         |
| Notification    | SNS (email alerts for processed uploads)      |
| Security        | IAM roles, policies, and bucket permissions   |

---

## **What This DevOps Accelerator Platform Covers**

### **Infrastructure Provisioning with Terraform**

* Fully automated infrastructure management.
* Remote backend configured using S3 for state files and DynamoDB for state locking.

### **End-to-End CI/CD with GitHub Actions**

* Automated workflows for:

  * Frontend deployment (S3 + CloudFront)
  * Backend Lambda build & deployment
  * Terraform provisioning
* Independent pipelines for each component.

### **Cloud-Native Hosting**

* Static frontend hosted on S3 and distributed via CloudFront CDN.
* Backend logic executed using AWS Lambda.
* Serverless, scalable, and cost-efficient architecture.

### **Secure File Upload Using Pre-Signed URLs**

* Users upload files securely without exposing the S3 bucket.
* Lambda functions are triggered automatically via S3 events.

### **AWS Monitoring & Alerting**

* CloudWatch logs track Lambda executions.
* SNS sends email notifications for successful file processing.
* Alerts provide better operational visibility.

### **Modular “Gigs” for Platform Extension**

* Easily extendable with plug-and-play modules like:

  * Project Generator
  * QA Bot
* Additional gigs can be added without impacting the core pipeline.

### **Scalable Folder Structure**

* Clean and modular folder layout separating infra, frontend, backend, and CI/CD workflows.
* Simple to reuse and replicate across AWS accounts.

---

## **Folder Structure**

```
DevOps-Accelerator-Project
├── .github
│   └── workflows
│       ├── backend-deploy.yml
│       ├── frontend.yml
│       └── terraform.yml
├── backend
│   └── lambda
│       ├── generate-presigned-url
│       │   ├── lambda.zip
│       │   └── main.py
│       └── process-uploaded-file
│           ├── lambda.zip
│           └── main.py
├── frontend
│   └── index.html
├── gigs
│   ├── project-generator
│   └── qa-bot
├── infra
│   └── terraform
│       ├── main.tf
│       ├── outputs.tf
│       ├── terraform.tfvars
│       └── variables.tf
└── README.md
└── .gitignore
```

---

## **Deployment Instructions**

### **1. Prerequisites**

* AWS CLI configured locally (`aws configure`)
* Terraform installed
* Node.js or Python (for Lambda packaging)
* GitHub repository for CI/CD

---

## **2. Infrastructure Setup Using Terraform**

Run the following commands inside the Terraform directory:

* `terraform init`

  * Initializes Terraform and configures backend.
* `terraform validate`

  * Checks if the configuration is correct.
* `terraform plan`

  * Shows what resources will be created.

### **Important Note:**

❗ Do **NOT** run `terraform apply` locally.
All changes must go through CI/CD — pushing to the main branch will trigger apply automatically.

---

## **3. CI/CD with GitHub Actions**

* The pipeline runs on every push to `main`.
* Automatically deploys:

  * Frontend updates
  * Lambda code changes
  * Terraform infrastructure updates
* Uses GitHub Secrets to authenticate AWS actions.

---

## **4. Deployment Execution Guidance**

```md
If you'd like to replicate this project (DevOps Accelerator) in your own AWS account, follow the steps below.
```

### **Prerequisites**

* AWS Account
* AWS CLI
* Git + GitHub
* Terraform v1.3+
* Node.js (optional future use)

---

### **Setting Up AWS Credentials**

1. Create an IAM User.
2. Generate an access key (download CSV).
3. Attach `AdministratorAccess` for setup purposes.
4. Configure locally:

```bash
aws configure
```

---

### **Clone the Repository**

```bash
mkdir my-devops-project && cd my-devops-project
git init
git clone git@github.com:Anees-DevOps/devops-accelerator.git .
```

---

### **Set Up Your GitHub Repository**

Create a new repository and add it:

```bash
git remote add origin git@github.com:your-username/your-repo.git
```

---

### **Add GitHub Repository Secrets**

Add secrets under:

**GitHub → Repo → Settings → Secrets → Actions**

| Secret Name           | Description                                  |
| --------------------- | -------------------------------------------- |
| AWS_ACCESS_KEY_ID     | IAM access key                               |
| AWS_SECRET_ACCESS_KEY | IAM secret key                               |
| AWS_REGION            | `us-east-1`                                  |
| LAMBDA_FUNCTION_NAME  | e.g., `process-uploaded-file`                |
| FRONTEND_BUCKET_NAME  | S3 bucket for frontend hosting               |
| UPLOAD_BUCKET_NAME    | S3 bucket for uploads                        |
| CLOUDFRONT_DIST_ID    | Added after first successful Terraform apply |

---

### **Prepare Lambda Deployments**

Re-zip backend functions before pushing:

```bash
cd backend/lambda/process-uploaded-file
zip -r lambda.zip .

cd ../generate-presigned-url
zip -r lambda.zip .
```

---

### **Manually Create Terraform State Backend**

```bash
aws s3api create-bucket \
  --bucket devops-accelerator-platform-tf-state \
  --region us-east-1

aws dynamodb create-table \
  --table-name devops-accelerator-tf-locker \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

---

### **Run Terraform Commands**

```bash
cd infra/terraform
terraform init
terraform validate
terraform plan
```

Push to start CI/CD:

```bash
git push -u origin main
```

---

### **Handling Git Push Errors**

For large file or `.terraform` folder errors:

```bash
echo ".terraform/" >> .gitignore
git rm -r --cached infra/terraform/.terraform
git commit -m "Remove .terraform directory"
```

If still failing:

```bash
git filter-repo --force --path infra/terraform/.terraform/ --invert-paths
git push --force
```

---

## **Validate Deployment**

CI/CD will automatically:

* Deploy frontend (S3 + CloudFront)
* Deploy backend Lambda functions
* Apply Terraform infrastructure

Terraform output example:

```
Outputs:
cloudfront_url = "d246o7opnvxl8.cloudfront.net"
frontend_bucket_name = "devops-accelerator-frontend-hosting-bucket"
lambda_function_name = "process-uploaded-file"
presigned_url_api_endpoint = "https://0jwmlx4c0a.execute-api.us-east-1.amazonaws.com"
```

Update placeholders with actual values and push again.

---

## **Verify AWS Resources**

You should now see:

* **3 S3 Buckets**
* **2 Lambda Functions**
* **1 API Gateway**
* **1 SNS Topic**
* **1 CloudFront Distribution**

---

## **Final Testing**

### **Open Website**

Use the CloudFront URL.

### **Upload a File**

Test with JPG/PNG/PDF.

### **Check CloudWatch Logs**

Validate Lambda processing.

### **SNS Notification**

You should receive a confirmation email.

---

## **You're Done! **

You've successfully deployed a **production-grade DevOps Accelerator** featuring:

* Automated infrastructure
* CI/CD pipelines
* Serverless backend
* S3 + CloudFront hosting
* CloudWatch monitoring
* SNS notifications

** Happy DevOps-ing! **

---
