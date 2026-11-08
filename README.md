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
3. Define AWS resources in CloudFormation with cost notes and a teardown path.
4. Add EKS, ECR, and Helm for the API; private S3 + CloudFront for static assets.
5. Add GitHub Actions with AWS OIDC role assumption—no long-lived AWS keys in GitHub.
6. Add CloudWatch baseline and optional Dynatrace telemetry.

## Safety and cost

- Never commit AWS credentials, local AWS profiles, `.env` files, or real customer data.
- Address fields currently remain in the browser page only; they are not transmitted or persisted. Avoid entering sensitive personal information.
- AWS deployment is not configured yet. EKS, NAT gateways, load balancers, logs, and observability can incur charges. Add billing alerts and review costs before deploying.
- Remove provisioned resources after practice sessions. Deleting CloudFormation stacks is part of the learning workflow.

## License

Personal learning project.
