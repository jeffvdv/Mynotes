# Automation

## Cloudformation

- Upload template in json or yaml to S3
- Sections:
    - Optional Description
    - Optional Metadata
    - Optional Parameters (f.e. Env type with allowed values prod stag)
    - Optional Conditions
    - Optional Mappings (f.e ami per region)
    - Optional Transform (to include template snippets for re-use in S3 or reference code for Lambda)
    - Mandatory Resources section
    - Optional Outputs
- You can rollback on failure (default it is checked)

## Elastic Beanstalk

Programming languages:
- Java
- .NET
- PHP
- Node.js
- Python
- Ruby
- Go
- Docker

Server platforms:
- Apache
- Tomcat
- Nginx
- Passenger
- IIS

Automatically updates OS, Java, PHP, ...

## OpsWork

- Puppet
- Chef

