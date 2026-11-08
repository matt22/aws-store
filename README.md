# Mare Tranquillitatis Supply Co.

A fictional lunar-surface outfitter and AWS/Kubernetes learning project. The shop sells practical equipment for people working, researching, and living on the Moon.

**Demo only:** checkout calculates an illustrative shipping estimate in the browser. It does not submit or retain the address, place a real order, take payment, fulfill, or ship anything.

## Run locally

Open `index.html` in a browser, or serve the repository root:

```sh
python3 -m http.server 8080
```

Then visit http://localhost:8080.

## Current scope

- Responsive storefront with product category filters and a cart.
- Browser-only simulated checkout with address-based shipping estimate.
- No backend, database, payment provider, or order persistence.
- Custom CSS product illustrations; Google Fonts are loaded from Google Fonts when online, with system fallbacks.

## Planned technical path

1. Finish the local storefront and its interaction flows.
2. Add a small API and container workflow where they support the learning goals.
3. Provision the static storefront with CloudFormation: a private S3 bucket and CloudFront distribution.
4. Add EKS, ECR, and Helm for the API; private S3 + CloudFront for static assets.
5. Add GitHub Actions with AWS OIDC role assumption—no long-lived AWS keys in GitHub.
6. Add CloudWatch baseline and optional Dynatrace telemetry.

## Safety and cost

- Never commit AWS credentials, local AWS profiles, `.env` files, or real customer data.
- Address fields currently remain in the browser page only; they are not transmitted or persisted. Avoid entering sensitive personal information.
- The AWS static-site template and manual GitHub Actions workflow are prepared but have not been deployed. EKS, NAT gateways, load balancers, logs, and observability can incur charges. Add billing alerts and review costs before deploying.
- Remove provisioned resources after practice sessions. Deleting CloudFormation stacks is part of the learning workflow.

## Static AWS deployment (manual)

The first AWS slice uses CloudFormation to create a private S3 bucket and a CloudFront distribution with Origin Access Control (OAC). S3 Block Public Access stays enabled; CloudFront is the only reader. The distribution uses the default CloudFront TLS certificate and the `PriceClass_100` edge locations. There is no custom domain, API, or customer data in this deployment.

### One-time setup

1. In AWS, create an IAM identity provider for `https://token.actions.githubusercontent.com` if the account does not already have one.
2. Create an IAM role for GitHub Actions with this trust policy, replacing `ACCOUNT_ID` and `GITHUB_ORG` as needed. Keep the repository and branch conditions narrow:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Principal": {"Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"},
       "Action": "sts:AssumeRoleWithWebIdentity",
       "Condition": {
         "StringEquals": {
           "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
           "token.actions.githubusercontent.com:sub": "repo:GITHUB_ORG/aws-store:ref:refs/heads/main"
         }
       }
     }]
   }
   ```

3. Attach a permissions policy to that role for the deployment. It needs CloudFormation stack create/update/describe, S3 sync/list access to the created site bucket, and CloudFront invalidation access to the created distribution. Scope S3 and CloudFront resources to this stack after its first creation; CloudFormation also needs permission to manage the bucket, bucket policy, OAC, and distribution resources. Review the policy before use.
4. In the GitHub repository settings, add the role ARN as the Actions secret `AWS_ROLE_ARN` and add `AWS_REGION` as a repository variable (for example, `us-west-2`). No AWS access keys are needed.

### Deploy

In GitHub, open **Actions → Deploy static site → Run workflow**. The workflow is manual; it provisions or updates the stack, syncs the storefront to S3, then invalidates CloudFront. It does not run on pushes. The distribution URL is shown in the workflow summary and CloudFormation output `SiteURL`.

### Costs and teardown

CloudFormation itself has no charge. S3 storage/requests and CloudFront requests/data transfer can incur charges; a deployed distribution remains provisioned between visits. Review current AWS pricing and account credits before deploying. This static slice avoids EKS, NAT gateways, and load balancers.

To tear down, disable the distribution and wait for CloudFormation to finish updating it:

```sh
aws cloudformation deploy \
  --template-file infra/static-site.yaml \
  --stack-name aws-store-static \
  --parameter-overrides DistributionEnabled=false \
  --region "$AWS_REGION"
```

Then empty the S3 bucket named in the stack output `SiteBucketName` and delete the `aws-store-static` CloudFormation stack. CloudFront distributions must be disabled before deletion, and a non-empty S3 bucket cannot be deleted by the stack. Confirm the stack is gone in CloudFormation and check Billing/Cost Explorer for any remaining usage charges.

## License

Personal learning project.
