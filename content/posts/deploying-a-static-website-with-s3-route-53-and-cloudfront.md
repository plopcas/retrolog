---
title: "Deploying a Static Website with S3, Route 53 and CloudFront"
date: 2020-02-09T18:55:52Z
draft: true
tags: ["software", "tools"]
author: "Pedro Lopez"
---

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront.jpg)

Today I'll show you how to deploy your static website to S3, and how you can configure a custom domain with Route 53 and enable HTTPS (why wouldn't you) with CloudFront. Keep reading to find out more!

<!--more-->

Even though this information is available in the AWS documentation, I ran into different problems when setting it up for my own website. Hopefully this runbook will help someone else as well.

#### Pre-requisites

1. An AWS account
2. A static website e.g. built with Hugo

Please note that Route 53 is not a free service. S3 and CloudFront are included in the AWS Free Tier within certain conditions. Check https://aws.amazon.com/free for more information.

Estimated costs for running a small static website are $10/year for the domain and $0.60/month for the running costs.

#### Getting a custom domain with Route 53

The first step is to get a new domain for our website. We'll do that with Route 53.

- Go to Route 53 https://console.aws.amazon.com/route53
- If it's your first domain, choose "Get Started Now" under "Domain Registration". If you already have another domain, in the navigation pane, choose "Registered Domains".

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-0.jpg)

- Choose "Register Domain".

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-1.jpg)

- Follow the steps and complete the registration. You will have to enter your contact details, verify your email address and make the payment.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-2.jpg)

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-3.jpg)

- Please note that the process can take a few days to complete.
- Once it's done, we can see a [Hosted Zone](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-working-with.html) has been automatically created for us. We will use this later on to configure our DNS records.

Source: https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html

#### Setting up S3 to serve static content

We will use the AWS console, but everything could also be done via CLI or CloudFormation.

Let's create the bucket first using the wizard.

- Create a new S3 bucket https://console.aws.amazon.com/s3.
- Give it a name that matches your domain name from the previous step e.g. example.com.
- Choose a region that is close to you and your potential users e.g. EU (London). This is to minimise latency, however, because we will be using CloudFront to serve our content, this is not really an issue. Another reason is to comply with relevant regulations for your country.
- Everything in the next step is optional, so you can skip it.
- Disable the "Block all public access" checkbox. As this is a bucket for a website, it needs to be public.
- Review and accept.

Next, let's configure the permissions so that the bucket is actually public. Note that in the previous step we disabled a checkbox that was preventing us to do this. This is important, becuase it's a misconception that disabling that checkbox is all you need to do. That's not correct, you also need to explicitely declare the bucket as public. For that, we'll use a policy.

- Go to the "Permissions" tab.
- Click on "Bucket policy".
- Create a new policy with the following (remember to replace example.com with your bucket name).
{{< highlight json >}}
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example.com/*"
        }
    ]
}
{{< /highlight >}}
- Go to "Properties".
- Click on "Static website hosting".
- Select "Use this bucket to host a website" and provide an index document e.g. `index.html`. This is the first page your website will load when a user lands on the root of the domain e.g. `https://example.com`.
- The error document is optional (but recommended).

Source: https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html

#### Setting up HTTPS with CloudFront

- Go to CloudFront https://console.aws.amazon.com/cloudfront
- Click on "Create distribution"
![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-.jpg)

Source: https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-https-requests-s3/

#### Joining the dots

#### (Hugo only) Deploying your website from CLI

---

{{< book link="https://www.amazon.co.uk/gp/product/1617295116/ref=as_li_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=1617295116&linkCode=as2&tag=retrolog-21&linkId=922f43e440dce0cd617f120fd471608c" image="//ws-eu.amazon-adsystem.com/widgets/q?_encoding=UTF8&MarketPlace=GB&ASIN=1617295116&ServiceVersion=20070822&ID=AsinImage&WS=1&Format=_SL250_&tag=retrolog-21" pixel="///ir-uk.amazon-adsystem.com/e/ir?t=retrolog-21&l=am2&o=2&a=1617295116" >}}
