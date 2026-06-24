# AWS-Static-Website-Hosting
A static website ("Code Tech IT Solutions") hosted on S3 and served globally through CloudFront over HTTPS. The S3 bucket stays fully private — CloudFront is the only thing allowed to read from it, enforced through a bucket policy scoped to this one specific distribution via Origin Access Control (OAC).
