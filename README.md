# Static_WebsiteHosting_Using_Terraform
To host a secure static website on AWS S3 using Terraform, you'll need to perform several steps:

1. **Create the S3 Bucket**: This is where your static website files will be stored.
2. **Configure the S3 Bucket for Static Website Hosting**: Enable website hosting on the bucket.
3. **Upload Website Content**: Upload the HTML, CSS, and JavaScript files to the S3 bucket.
4. **Set up an SSL/TLS Certificate (Optional, for HTTPS)**: You can secure your website by using AWS Certificate Manager (ACM) and CloudFront to serve your content securely.
5. **Configure CloudFront Distribution** (Optional): This will allow you to use HTTPS and provide caching and CDN features.

Here’s a step-by-step guide on how to do this with Terraform.

### 1. Set up Terraform Configuration File

#### Provider Setup

Create a `main.tf` file with your AWS provider configuration:

```hcl
provider "aws" {
  region = "us-east-1"  # Set your desired AWS region
}

resource "aws_s3_bucket" "static_website" {
  bucket = "my-static-website-bucket"  # Choose a globally unique name for the bucket

  website {
    index_document = "index.html"
    # Optional error document
    # error_document = "error.html"
  }
}

# Optional: Configure CloudFront distribution for HTTPS (use SSL)
resource "aws_acm_certificate" "website_cert" {
  domain_name = "www.mydomain.com"  # Replace with your domain
  validation_method = "DNS"
}

resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.static_website.website_endpoint
    origin_id   = "S3-my-static-website-bucket"

    s3_origin_config {
      origin_access_identity = "origin-access-identity/cloudfront/EXAMPLE"
    }
  }

  enabled = true
  is_ipv6_enabled = true

  default_cache_behavior {
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods {
      methods = ["GET", "HEAD"]
    }
  }

  viewer_certificate {
    acm_certificate_arn = aws_acm_certificate.website_cert.arn
    ssl_support_method   = "sni-only"
  }
}

output "website_url" {
  value = aws_s3_bucket.static_website.website_endpoint
}
```

### 2. Create the S3 Bucket and Enable Static Website Hosting

In the code above, the `aws_s3_bucket` resource will create the S3 bucket and enable static website hosting with an `index.html` file. You can also define an error page if you want.

### 3. Configure DNS (Optional but Recommended for Custom Domains)

You can use Amazon Route 53 to set up DNS records that point to your S3 bucket or CloudFront distribution.

#### DNS Record Setup:

```hcl
resource "aws_route53_record" "www" {
  zone_id = "YOUR_ZONE_ID"  # Your Route 53 hosted zone ID
  name    = "www.mydomain.com"  # Replace with your domain
  type    = "A"
  alias {
    name                   = aws_cloudfront_distribution.cdn.domain_name
    zone_id                = aws_cloudfront_distribution.cdn.hosted_zone_id
    evaluate_target_health = true
  }
}
```

### 4. Secure Website with HTTPS (Optional)

To ensure your static website is served securely, use AWS CloudFront with an SSL/TLS certificate from AWS Certificate Manager (ACM). This is already included in the above example.

- Ensure that the domain you want to use is validated in ACM.
- Use the ACM certificate with CloudFront.

### 5. Run Terraform Commands

Once your `main.tf` file is ready, follow these steps:

1. **Initialize Terraform**:

```bash
terraform init
```

2. **Plan the Changes**:

```bash
terraform plan
```

3. **Apply the Configuration**:

```bash
terraform apply
```

Confirm the action when prompted.

### 6. Upload Files to S3

After Terraform has created the resources, you can manually upload your static website files to the S3 bucket, or you can use Terraform's `aws_s3_bucket_object` resource to automate this.

Example for uploading files:

```hcl
resource "aws_s3_bucket_object" "index" {
  bucket = aws_s3_bucket.static_website.bucket
  key    = "index.html"
  source = "path/to/your/local/index.html"
  acl    = "public-read"
}
```

### 7. Access Your Static Website

After everything is set up, you can access your website:

- If you're using S3 without CloudFront, you can use the `website_endpoint` output value from Terraform to access the site, e.g., `http://my-static-website-bucket.s3-website-us-east-1.amazonaws.com`.
- If you're using CloudFront and SSL, access the site via `https://www.mydomain.com`.

### Conclusion

This Terraform script sets up an S3 bucket for static website hosting and can be secured with HTTPS using CloudFront and ACM. By applying this, you'll have a scalable and secure static website hosting solution.
