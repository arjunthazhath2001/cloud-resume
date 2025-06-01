# 🚀 Hosting a Single Page Resume on AWS using S3, CloudFront & GitHub Actions

## ✅ What I Did

## DEMO VIDEO:
<iframe width="560" height="315" src="https://www.youtube.com/embed/6SaSbUW_eGM?si=d_fwmRkyDCF9xfrq" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### 1. Created an S3 Bucket

* **Region**: `ap-south-2`
* Enabled **Static Website Hosting**
* Disabled **Block All Public Access**
* Added a **Bucket Policy** to:

  * Allow read (`s3:GetObject`) for CloudFront
  * Allow write/delete (`PutObject`, `DeleteObject`) from a GitHub Actions IAM user

### 2. Issued an SSL Certificate (ACM)

* **Region**: `us-east-1` (Required for CloudFront to recognize it)
* Validated domain and subdomain using **CNAME records** in Route 53

### 3. Created a CloudFront Distribution

* **Origin**: The *S3 static website endpoint*
* **Default Root Object**: `index.html`
* **Alternate Domain Names (CNAMEs)**: e.g., `resume.arjunta.tech`
* **Custom SSL Certificate**: From ACM (`us-east-1`)
* Set up **cache behavior** to forward minimal headers for static content

### 4. Created a Public Hosted Zone in Route 53

* **Domain**: `arjunta.tech`
* Copied **NS records** to Hostinger's nameserver settings
* Created an **Alias Record** for `resume.arjunta.tech` → CloudFront Distribution

### 5. Provisioned a Dedicated IAM User for CI/CD

* Attached a custom IAM policy to:

  * Upload/delete files in S3
  * Invalidate CloudFront cache (`cloudfront:CreateInvalidation`)
* Followed **principle of least privilege** (did not use root credentials)

### 6. Stored IAM Credentials in GitHub Secrets

* `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `DISTRIBUTION`

### 7. Configured GitHub Actions

* **Trigger**: `on: push` to `main` branch
* **Steps**:

  1. `actions/checkout@v3`
  2. `aws-actions/configure-aws-credentials@v2`
  3. `aws s3 sync . s3://bucket-name --delete`
  4. CloudFront cache invalidation via: `chetan/invalidate-cloudfront-action@v2` (3rd-party action)
