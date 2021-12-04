# Amazon Macie

- It is a data security and data privacy service
- Macie is a service to discover, monitor and protect data stored in S3 buckets
- Once enabled and pointed to buckets, Macie will automatically discover data and categorize it as PII, PHI, Finance etc.
- Macie is using a data identifier. There are 2 types of data identifier:
    - **Managed Data Identifier**: built-in, can use machine learning, pattern matching to analyze and discover data. It is designed to detect sensitive data from many countries
    - **Custom Data Identifier**: created by clients, they are proprietary to accounts and they are regex based
- Discovery Jobs: these jobs will use data identifiers to manage and search for sensitive content. They will generate findings which can be used for integration with other AWS service (ex: Event Bridge) in order to do automatic remediation
- Macie uses multi account architecture: one account is the master account which can manage other accounts to discover sensitive data