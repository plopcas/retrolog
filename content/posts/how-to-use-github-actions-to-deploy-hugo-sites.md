---
title: "How to use GitHub Actions to Deploy Hugo Sites"
date: 2020-08-23T22:09:48+01:00
tags: ["software", "tools"]
draft: false
---

![image](/images/how-to-use-github-actions-to-deploy-hugo-sites.jpg)

I have created a new GitHub Action that deploys to S3 using a configured deployment target in Hugo. Additional information about Hugo deployments is available in the [official documentation](https://gohugo.io/hosting-and-deployment/hugo-deploy/).

<!--more-->

#### Why?

I use Hugo for my personal website, and every time I make a change or create new content I need to build the site locally and then deploy it. But not any more. Now, with every commit, GitHub will build the site, deploy it to S3 and invalidate my CloudFront distribution so that fresh new content is available immediately.

#### Are there similar actions out there?

Yes and no. I explored the Marketplace and could find workarounds. For example, there are actions to build a Hugo site, and there are actions to deploy it manually by uploading the output folder (`public` typically) to a remote destination, like S3. And there are actions to invalidate CloudFront distributions.

However, Hugo already provides a mechanism to do all this for you, the `hugo deploy` command. Yet, there was no GitHub action I could find to run your this command and execute your configured deployment.

This action solves this.

#### Implementation details

I have created a Docker Container action for this. Documentation available [here](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action).

##### Dockerfile

To define my container I use a Dockerfile.

```
FROM pahud/awscli-v2:node-lts

RUN yum update -y && \
    yum install -y curl jq

COPY entrypoint.sh /

ENTRYPOINT ["/entrypoint.sh"]
```

I'm using a base image that contains AWS CLI already, so that I can perform the CloudFront invalidation. Reference at the bottom.

Then I'm installing `curl` and `jq` which I'll use to install Hugo in the container later on.

Finally, I specify the entrypoint.

##### entrypoint.sh

Important to remember to make it executable as explained in the documentation.

```
chmod +x entrypoint.sh
```

The entrypoint is what contains the logic that will be executed when the action runs.

First, fail the pipeline immediately if there are any errors.

```
set -eo pipefail
```

Check the different parameters that we pass as environment variables. E.g.:

```
if [ -z "$AWS_ACCESS_KEY_ID" ]; then
  echo "error: AWS_ACCESS_KEY_ID is not set"
  err=1
fi
```

Create an AWS profile for this action.

```
aws configure --profile hugo-s3 <<-EOF > /dev/null 2>&1
${AWS_ACCESS_KEY_ID}
${AWS_SECRET_ACCESS_KEY}
${AWS_REGION}
text
EOF
```

Then install Hugo by fetching the latest version with curl (not adding the code here, but can be found in the repo).

Build and deploy the site.

```
hugo
hugo deploy
```

And finally, we clean up the AWS profile (not really needed if the container is destroyed immediately afterwards but good practice to clean up after ourselves).

##### action.yml

Contains the metadata for the action.

```
name: 'Hugo S3'
description: 'Deploy Hugo with an S3 target'
author: 'Pedro Lopez'
branding:
  icon: 'book'
  color: 'purple'
runs:
  using: 'docker'
  image: 'Dockerfile'
```

### Submission Category

DIY Deployments

### Yaml File or Link to Code

This action is now available in the GitHub Marketplace 
https://github.com/marketplace/actions/hugo-s3.

{% github https://github.com/plopcas/hugo-s3-action %}

I've also configured it on my personal site https://retrolog.io/, which is also publicly available on GitHub.

An example of a successful run is available here 
https://github.com/plopcas/retrolog/runs/1018289670.

**Usage**

```
name: Hugo S3

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    
    runs-on: ubuntu-latest

    steps:
      - name: Check out master
        uses: actions/checkout@master
          
      - name: Deploy site
        uses: plopcas/hugo-s3-action@v1.3.0
        env:
          AWS_REGION: 'eu-west-2'
          AWS_ACCESS_KEY_ID: ${{ secrets.ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.ACCESS_KEY_SECRET }}
```

### Additional Resources / Info

GitHub repository https://github.com/plopcas/hugo-s3-action
Personal site on GitHub https://github.com/plopcas/retrolog

#### References

GitHub actions documentation https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action
Hugo on GitHub https://github.com/gohugoio/hugo
AWS CLI Docker base image https://hub.docker.com/r/pahud/awscli-v2
