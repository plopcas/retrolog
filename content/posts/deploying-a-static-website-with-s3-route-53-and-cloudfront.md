---
title: "Deploying a Static Website with S3, Route 53 and CloudFront"
date: 2020-02-09T18:55:52Z
draft: false
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

Estimated costs for running a small static website are $12/year for the domain and $0.60/month for the running costs.

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

- Please note that the process can take up to a few days to complete.
- Once it's done, we can see a [Hosted Zone](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-working-with.html) has been automatically created for us. We will use this later on to configure our DNS records.

#### Setting up S3 to serve static content

We will use the AWS console, but everything could also be done via CLI or CloudFormation.

Let's create the bucket first using the wizard.

- Create a new S3 bucket https://console.aws.amazon.com/s3.
- Give it a name that matches your domain name from the previous step e.g. example.com.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-4.jpg)

- Choose a region that is close to you and your potential users e.g. EU (Ireland). This is to minimise latency, however, because we will be using CloudFront to serve our content, this is not really an issue. Another reason is to comply with relevant regulations for your country.
- Everything in the next step is optional, so you can skip it.
- Disable the "Block all public access" checkbox. As this is a bucket for a static website, it needs to be public. Please note that we haven't made the bucket public yet, we just removed the blocker that prevented us from doing so.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-5.jpg)

- Review and accept.

Next, let's configure the bucket for static website hosting.

- Go to "Properties".
- Click on "Static website hosting".

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-6.jpg)

- Select "Use this bucket to host a website" and provide an index document e.g. `index.html`. This is the first page your website will load when a user lands on the root of the domain e.g. `https://example.com`. The error document is optional (but recommended).

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-7.jpg)

- Make a note of the "Endpoint" URL at the top e.g. `http://example.com.s3-website-eu-west-1.amazonaws.com/` and click "Save".

Let's test things by uploading an `index.html` file to the bucket.

- Create a file named `index.html` with your favourite text editor or IDE and paste the following.

{{< highlight html >}}
<html>
    <body>
        <span>Hello world!</span>
    </body>
</html>
{{< /highlight >}}

- Go to the "Overview" tab and click on "Upload", then select the `index.html` file you just created.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-8.jpg)

- Click the "Upload" button at the bottom to skip all the additional steps and leave the defaults.
- Open the "Endpoint" URL from the previous step in a browser.

A **403 Forbidden** error!? 😱 What's going on?

What happened is that the file itself is not public yet. We can fix this!

- Go back to the "Overview" tab and click on the `index.html` file.
- On the new page, click on the "Permissions" tab.
- At the bottom, click on "Everyone".

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-9.jpg)

- Select "Read object" like in the image.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-10.jpg)

- Open again the "Endpoint" URL from the previous step in a browser (or refresh the tab).

**Hello world!** 🎉

We could have made the object public when we uploaded it as well, but we skipped that step and left all the defaults to make a point.Either way, marking each object as public individually is not a very efficient way to manage permissions. Let's configure the bucket so that all its objects are public at once. For that, we'll use a policy.

- Go back to the bucket itself using the breadcrumb menu at the top.
- Go to the "Permissions" tab.
- Click on "Bucket Policy".
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

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-11.jpg)

- If we open the "Endpoint" URL once again...

**Hello world!** 🎉💃🕺🎉

#### Setting up a custom domain

At the beginning of this post, we bought a new custom domain e.g. example.com, but we are still accessing our website with a very long Amazon looking URL like `http://example.com.s3-website-eu-west-1.amazonaws.com/`. We don't want that, we want to use `http://example.com`.

- Go to Route 53 https://console.aws.amazon.com/route53
- Click on "Hosted Zones" and select the one that matches your domain name.
- Click on the "Create Record Set" button at the top.
- On the form, leave the *Name* box empty to use the default value e.g. example.com, and make sure IPv4 is selected.
- Select Alias "Yes", click inside the input box and select your S3 bucket matching your domain name e.g. example.com.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-12.jpg)

- Open your domain e.g. `http://example.com`.

**Hello world!** 🎉💃🕺💃🕺💃🕺💃🕺🎉

We have our static (super simple) website running in a custom domain. However, we are still using http and not https, not good. Let's change that.

#### Setting up HTTPS with CloudFront

- Go to CloudFront https://console.aws.amazon.com/cloudfront
- Click on "Create distribution".
- On the next step, select Web as your delivery method by clicking the "Getting Started" button at the top.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-13.jpg)

- On the next step, for "Origin Domain Name" insert your "Endpoint" URL without the `http://` e.g. `example.com.s3-website-eu-west-1.amazonaws.com`. 🚨 **THIS IS VERY IMPORTANT!!!!** 🚨 If you just select S3 from the drop down, by default it will autocomplete with something like `example.com.s3.amazonaws.com` and you will get an `Access denied` exception when accessing your website with HTTPS. This is because the latter is the REST format, but we need the website format, more information [here](https://aws.amazon.com/premiumsupport/knowledge-center/s3-rest-api-cloudfront-error-403/).

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-14.jpg)

- Select "Redirect HTTP to HTTPS" to make sure encryption its always used.
- Still on the same step, scroll down to the bottom of the form.
- Inside the "Alternate Domain Names (CNAMEs)" text area, enter your domain name e.g. example.com.
- For SSL Certificate, choose "Custom SSL Certificate".

Wait... it's grayed out!? 😱 We need an SSL certificate that we don't have. For that we'll use Amazon ACM. There is a convenient button that says Request or Import a Certificate with ACM. This will open a new tab. Alternatively, you can navigate to ACM using the "Services" menu at the top of the AWS console as usual.

- Make sure that you are in the **North Virginia** region. 🚨 **THIS IS VERY IMPORTANT!!!!** 🚨 It's the only region supported at the moment (2020), otherwise you will not be able to import the certificate in CloudFront.
- Enter your domain name e.g. example.com.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-15.jpg)

- Choose DNS validation.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-16.jpg)

- Skip the next steps until you get to "Review" and click "Confirm and request"
- In the "Validation" step, click on the arrow next to your domain name to explan, and click on "Create record on Route 53".

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-17.jpg)

- In the pop-up window click "Create". You'll see a warning message that saying that it may take up to 30 minutes for the changes to propagate, and for AWS to validate the domain. If we go back to our Hosted Zone in Route 53 we'll see that a new CNAME entry has been created.
- Grab a cup of your favourite beverage and wait ☕️

Once the certificate is ready, we can go back to where we left in CloudFront. It should now be possible to choose a "Custom SSL Certificate". Refresh the page if it's not the case, you might need to re-enter the information in the form.

- Back to CloudFront, select the recently created SSL certificate.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-18.jpg)

- Leave everything else with the default values and click "Create Distribution" at the bottom.

#### Joining the dots

We have all the ingredients, but there is one last step, we need to configure our DNS records to use CloudFront instead of S3.

- Go to Route 53 https://console.aws.amazon.com/route53
- Navigate to your Hosted Zone.
- Select the alias that is pointing to S3 (Type A).
- On the form that shows up, click on the input field, delete its value and wait for the autocomplete to come up. Then scroll down and select your CloudFront distribution.

![image](/images/deploying-a-static-website-with-s3-route-53-and-cloudfront-19.jpg)

- Save and after a few minutes, navigate to your website using HTTPS e.g. `https://example.com` (replace with your domain name).

```
🎉🎉🎉🎉🎉🎉🎉🎉
🎉Hello world! 🎉
🎉🎉🎉🎉🎉🎉🎉🎉
````

#### Read more

There is official documentation that covers this use case, however it's split between a few documents. Hopefully I've summarised all the steps in this post, but if you want to know more, you can check the Amazon guidelines.

- [Domain register](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html)
- [Website hosting custom domain walkthrough](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)
- [CloudFront HTTPs requests and S3](https://aws.amazon.com/premiumsupport/knowledge-center/cloudfront-https-requests-s3/)

Thanks for reading!

---

{{< book link="https://www.amazon.co.uk/gp/product/1617295116/ref=as_li_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=1617295116&linkCode=as2&tag=retrolog-21&linkId=922f43e440dce0cd617f120fd471608c" image="//ws-eu.amazon-adsystem.com/widgets/q?_encoding=UTF8&MarketPlace=GB&ASIN=1617295116&ServiceVersion=20070822&ID=AsinImage&WS=1&Format=_SL250_&tag=retrolog-21" pixel="///ir-uk.amazon-adsystem.com/e/ir?t=retrolog-21&l=am2&o=2&a=1617295116" >}}
