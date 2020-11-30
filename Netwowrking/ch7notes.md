# Design and implement for Security and Compliance

## Traffic Control

###AWS Shield:
- DDos protection
- Hosted cloudfront, Global Acceleration and Route53 Edge locations
- Covers 96% of known layer 3 and 4 attacks

2 pricings:
- Standard tier (automatically)
- Advanced tier:
    - Additional protection
    - Detailed monitoring
    - WAF no charge
    - 27/7 DDos response team
    - EDos (economic denial of sustainability) coverage (when scaling costs occur)

###WAF:
- Allow/Deny HTTP/HTTPS using ACL
- Applicable to API Gateway, Cloudfront, ALB
- newer version has been released (WAF and WAF classic)

Conditions:
- Cross-site scripting
- Country of origin
- IP address
- Size of request properties
- SQL queries
- String/Regex

Each condition contains one or more filters.
Multiple conditions are OR-ed

Rules:
- One or more conditions
- Specify match or not match
- Multiple conditions are AND-ed
- normal/rate-based (cloudwatch)

Customers may purchase managed rule-sets

ACL:
- one or more rules
- allow/deny/count
- Sequential order (count exception will only count, other acls still apply till match)
- Default actions

Price:
- Per ACL
- Per Rule

Limits per account/per region:
- 50 ACLS
- 100 Rules
- 5 rate-based rules
- 100 of each condition type except
- 10 regex conditions (cannot be increased)

Maximum 10 conditions per rule

Maximum 10 rules per ACL

###Cloudfront

###ALB

Authentication can be offloaded to the ALB
- Traffic matching a listener rule is send to a configured identity provider
