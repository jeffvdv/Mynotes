# Updates based on Student Feedback

## Service Catalog

- Create product (like create private s3 Bucket defined in Cloudformation Template)
- Create portfolio
- Add product to portfolio
- Create user 
    - allow access to the portfolio (AWSServiceCatalogEndUserFullAccess)
    - allow s3 bucket access (S3FullAccess)
- Add user to portfolio
- You can share portfolios with other AWS accounts

## Cloudfront Error Messages

- 400 => Malformed request
- 403 => Access denied, files must be accesible f.e. s3
- 404 => File not found
- 502 => Bad Gateway, Cloudfront cant connect to server
- 503 => Service unavailable, performance issue on the server
- 504 => Gateway Timeout, Request timeout

## Multi-account to Direct Connect

up to 10 VPC and same payer account

## Inter-regin VPC peering

VPC peering request, change routing table, can be done in 2 accounts in different regions.
CIDR ranges need to be unique.

## HTTPS & Storing SSL Certificates

- Certificate Manager
- Free
- Upload existing certificates

## Cloudformation Best Practices

- IAM
- Be aware of service limits
- Avoid manual updates => chance of mismatch
- Use Cloudtrail to monitor the changes
- Stack policy (describe update policy on critical resources f.e. production database)


