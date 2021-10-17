# S3 Access Points

- Improves the manageability of objects when buckets are used for many different teams or they contain objects for a large amount of functions
- They simplify the process of managing access to S3 buckets/objects
- Rather than 1 bucket (1 bucket policy) access we can create many access points with different policies
- Each access point can be limited from where it can be accessed, and each can have different network access controls
- Each access point has its own endpoint address
- We can create access point using the console or the CLI using `aws s3control create-access-point --name < name > --account-id < account-id > --bucket < bucket-name >`
- Any permission defined on the access point needs to be defined on the bucket policy as well