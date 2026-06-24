# AWS Static Website Hosting

**Stack:** Amazon S3 + Amazon CloudFront (Origin Access Control)

---

## What this is

A static website ("Code Tech IT Solutions") hosted on S3 and served globally through CloudFront over HTTPS. The S3 bucket stays fully private — CloudFront is the only thing allowed to read from it, enforced through a bucket policy scoped to this one specific distribution via Origin Access Control (OAC).

```
Visitor → CloudFront (HTTPS, caching, OAC) → S3 bucket (private origin)
```

No public bucket, no S3 website-endpoint URL in use — just OAC-authenticated reads from a locked-down bucket.

---

## Step 1: Create the S3 bucket

Created `codetech-it-solutions-intership` in US East (Ohio), with all four Block Public Access settings left on.

![S3 bucket created](images/01-s3-bucket-created.png)

---

## Step 2: Create the CloudFront distribution

Distribution `E2BSQS7MEAETPR` created (Standard type), origin pointed at the S3 bucket, status flipped to **Enabled**. Default root object set to `index.html`, with `d3fw7tzxibtqht.cloudfront.net` assigned as the distribution domain.

![CloudFront distribution enabled](images/02-cloudfront-distribution-enabled.png)
![CloudFront distribution settings](images/04-cloudfront-distribution-settings.png)

---

## Step 3: Upload the website files

`index.html` and `error.html` uploaded to the bucket.

![Files uploaded to S3](images/03-files-uploaded-to-s3.png)

---

## Step 4: Lock the bucket to CloudFront only

Bucket policy applied with a `Condition` scoped to this exact distribution's ARN — only `Cloudfront-distribution` can read from this bucket, nothing else.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::codetech-it-solutions-intership/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::Account-ID:distribution/Cloudfront-distribution"
        }
      }
    }
  ]
}
```

![S3 bucket policy with OAC](images/05-s3-bucket-policy-oac.png)

---

## Step 5: Test the live site

Loaded the CloudFront domain and scrolled through the whole page — hero section, services, process steps, stats, and the contact form at the bottom.

![Live site — hero section](images/06-live-site-hero.png)
![Live site — services section](images/07-live-site-services.png)
![Live site — process section](images/08-live-site-process.png)
![Live site — stats and contact](images/09-live-site-stats-contact.png)
![Live site — contact form and footer](images/10-live-site-contact-footer.png)
![Live site — reload test](images/11-live-site-reload-test.png)

---

## Step 6: Check CloudFront monitoring

Confirmed requests were actually hitting the distribution and that the error rate stayed at 0%.

![CloudFront monitoring — requests](images/12-cloudfront-monitoring-requests.png)
![CloudFront monitoring — data transfer](images/13-cloudfront-monitoring-data-transfer.png)
![CloudFront monitoring — error rate](images/14-cloudfront-monitoring-error-rate.png)

---

## Step 7: Review the CloudFront console layout

Took a pass through the left-hand navigation — Distributions, Policies, Functions, Telemetry (Monitoring/Alarms/Logs), Reports & analytics, Security — to get oriented for later projects that'll touch these same sections.

![CloudFront console navigation](images/15-cloudfront-console-navigation.png)

---

## Step 8: Confirm the public-website route stayed off

Double-checked S3's built-in static website hosting feature is **Disabled** — the site is served only through CloudFront + OAC, never through S3's own public website endpoint.

![S3 static website hosting disabled](images/16-s3-static-hosting-disabled.png)

---

## Notes / what this taught me

- **OAC over a public bucket.** Keeping Block Public Access fully on and using OAC instead means the bucket is never exposed directly — CloudFront is the only path in, and it's locked to one specific distribution ARN.
- **`Default root object` is easy to forget.** Without setting it to `index.html`, the root URL just returns nothing useful.
- **CloudFront takes a few minutes to deploy.** Status sits on "Deploying" before flipping to "Enabled" — worth checking before assuming something's broken.
- **Monitoring confirmed the loop end-to-end.** Seeing real request counts and a 0% error rate in CloudFront's metrics was the actual proof the whole chain (bucket → policy → distribution → viewer) was wired correctly, not just "the page loaded once."
