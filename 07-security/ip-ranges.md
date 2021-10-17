# `ip-ranges.json`

- It is a json file which contains structures for the public IP ranges of every service of every AWS region
- The `ip-ranges.json` file is updated whenever a new IP range is added or removed
- This file is maintained by AWS and it is hosted publicly: [https://ip-ranges.amazonaws.com/ip-ranges.json](https://ip-ranges.amazonaws.com/ip-ranges.json)
- An AWS managed SNS topic also receives a notification whenever this file is updated. This topic is also maintained by AWS and anyone can subscribe
- We can use the IP ranges information to manually update security groups
- We can also parse the file with a script an automatically update security configurations at scale
- We can use an event driven flow invoking a Lambda function when we receive a notification from the SNS topic if the file was updated