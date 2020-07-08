# Security & Compliance

## Compliance on AWS

- FedRAMP
    - US
- ISO
    - ISO/IEC 27001:2005 Documented Information Security Management System
- HIPAA
    - Health Insurance Protability and Accountability Act (storage)
- NIST
    - US Cybersecurity
- PCI (Credit card information)
    1. Install and maintain firewall configuration to protect cardholder data
    2. Do not use vendor-supplied defaults for system passwords and other security parameters
    3. Protect and stored cardholder data (like encrypt database at rest)
    4. Encrypt transmission of cardholder data across open, public networks (SSL)
    5. Protect all systems against malware and regularly update anti-virus software
    6. Develop and maintain secure systems and applications
    7. Restrict access to cardholder data by business need to know (sys admins do not need to know the data)
    8. Identity and authenticate access to system components (IAM)
    9. Restrict physical access to cardholder data (do not copy creditcards on paper f.e)
    10. Track and monitor all access to network resources and cardholder data (Cloudwatch, Cloudtrail, Config)
    11. Regularly test security systems and processes (Penetration testing)
    12. Maintain a policy that addresses information security for all personal
- SAS70 (Auditing standards)
- SOC1 (Service Organisation Controls - accounting standards)
- FISMA (Federal Information Security Modernization Act)
- FIPS (FIPS 140-2 cryptographic modules - CloudHSM is level 3)
  
## DDoS

- NTP Amplification - Hacker sends packets with spoofed IP address source to the ntp server which replies with a 
greater payload to the victim (spoofed ip).
- Application Attacks - Flood of GET requests to webserver
- Slowloris - Hacker sends partial requests to the webserver to keep op as many connections open as possible.
This keeps up until the concurrent connections pool is filled.

### Mitigate DDoS

- Minimize the Attack Surface Area (WAF and ALB)
- Be ready to scale (Autoscaling groups)
- Safeguard Exposed Resources
- Learn Normal Behavior
- Create plan for attacks

### AWS Shield

- Free service that protects all AWS customers on ELB, Cloudfront and Route53
- Protects against SYN/UDP Floods, Reflection attacks, and other layer 3/ layer 4 attacks.
- Advanced:
    - provides protection against larger and more sophisticated attack
    - $3000 per month
    - Always onm flow-based monitoring of network traffic and active application monitoring
    to provide near real-time notifications of DDoS attacks.
    - DDoS Response Team (DRT)
    - Protects your AWS bill against higher fees during DDoS attack.
    
## AWS Marketplace - Security Products

- Kali Linux
- Ask permission to AWS for penetration testing (request form)

## MFA & Reporting with IAM

In IAM you can select user and enable MFA.

```yaml
aws iam create-virtual-mfa-device --virtual-mfa-device-name EC2-user --outfile ./QRCode.png --bootstrap-method QRCodePNG
aws iam enable-mfa-device --user-name EC2-user --serial-number arn:aws:iam::[username]:mfa/EC2-user \
--authentication-code-1 [code1] --authentication-code-2 [code2]
```

Look into IAM credentials report to verify users have MFA enabled.

## Security Token Service (STS)

- Federation (typically Active Directory)
    - SAML
    - Temporary Access based of the users Active Directory credentials
    - SSO without assigning IAM credentials
- Federation with Mobile Apps
    - Use Facebook/Amazon/Google or other OpenId providers
- Cross Account Access

### Flow
![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Screenshot%2020-07-08%at%07.53.34.png)

## Logging

- AWS Cloudtrails
- AWS Config
- AWS Cloudwatch Logs
- VPC Flow Logs

### Control Access to Log Files

Prevent unauthorized access:

- IAM users, groups, roles and policies
- Amazon S3 bucket policies
- Multi Factor Authentication

Ensure role-based access:

- IAM users, groups, roles and policies
- Amazon S3 bucket policies

Alerts when logs are created or failed:

- Cloudtrail notifications
- AWS Config Rules

Alerts are specific, but don't divulge detail:

- Cloudtrail SNS notifications only point to log file location.

Log changes to system components:

- (AWS Config Rules)
- Cloudtrail

Controls exist to prevent modifications to logs:

- IAM and S3 controls and policies
- CloudTrail log file validation
- CloudTrail log file encryption

## AWS WAF

- Allow all requests except the ones that you specify
- Block all requests execpt the ones that you specify
- Count the requests that match the properties that you specify

What you specify:

- originated ip addresses
- originated country
- Values in headers
- Strings via regex
- Length of requests
- Presence of SQL code (SQL injection)
- Presence of Script (cross-site-scripting)

WAF integrates with:

- ALB
- Cloudfront
- API Gateway

NOT: Classic loadbalancer or network loadbalancer (not Layer 7)




