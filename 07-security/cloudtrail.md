 # CloudTrail

- Is a product which logs API actions which affects AWS accounts (example: stop EC2 instance, delete S3 bucket)
- It logs API calls/activities as a CloudTrail Event, actions taken by user, role or service
- CloudTrail by default logs the last 90 days in Event History. It is enabled by default at no cost
- Trails: used to customize CloudTrail history
- Trails can be of 3 types:
    - **Management Events**: provide information about management operations performed on resources in AWS accounts (control plane operation). Example: create an EC2 instance, terminate an EC2 instance
    - **Data Events**: contain information about resource operations performed on or in a resource, example: objects uploaded to S3, Lambda function being invoked
    - **Insights Events**: CloudTrail Insights analyzes our normal patterns of API call volume and API error rates, also called the *baseline*, and generates Insights events when the call volume or error rates are outside normal patterns
- By default CloudTrail only logs management events
- CloudTrail is a regional service, but when we create a Trail, it can be configured to operate in two ways:
    - One region: only will log events from the region in which it is created
    - All regions: collection of trails from all regions
- A one region trail can be configured to log global services events as well
- Most services log events in the region where the event occurred, but a small number of services (IAM, STS, CloudFront) log events globally (us-east-1). For a trail to accept global events, it has to be all region trail (has to be enabled for the trail)
- The default event history is limited to 90 days, with a trail we can be much more flexible
- A trail can store the events in a defined S3 buckets indefinitely, and these logs can be parsed by other tooling (log entries are stored in JSON format)
- CloudTrail can be integrated with CloudWatch Logs, where we can use Metric Filters for example
- Organizational Trail: 
    - A trail created in the management account of an organization storing all events across every account in the organization
    - In case we create a trail from the management account of an organization, we can enable it to bring events from all of the accounts from the organization
- CloudTrail does not offer real time logging!
- CloudTrail pricing:
    - 90 days history enabled by default in every AWS is free
    - One copy of management events is free for every region in an AWS account
    - Other trails created are charged

## Log File Integrity Validation

- For many organization for every CloudTrail log file we receive we need to know if those files are valid
- We can verify the validity of each log file with the Log File Validation feature offered by CloudTrail
- This feature can be enabled on a custom trail, once enable CloudTrail will create and sign digest files. Digest files are delivered in the same S3 bucket as the logs, but in a different folder
- Digest files contain a hash of every log file delivered in the last hour
- Using the digest files we can identify if there is a log file missing. We can also identify is a log file was edited
- How can we trust a digest file:
    - Each digest file contains the signature of a previous digest file
    - CloudTrail signs each digest file with a private key. We can use the corresponding public key to validate the authenticity of a digest file