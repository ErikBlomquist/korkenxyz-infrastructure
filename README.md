# korkenxyz-infrastructure
- Terraform
- S3
- CloudFront
- Travis CI

1. Change the following into your own domains before running `terraform apply`.
```
# Variable for domain name
variable "www_domain_name" {
  default = "www.korken.xyz"
}

# Variable for root domain
variable "root_domain_name" {
  default = "korken.xyz"
}
```

2. Make sure that you have access to one of the following e-mail addresses associated with your domain. I personally registered a free Zoho account that allowed me to create an e-mail address for my domains purchased from GoDaddy.

    - administrator@your_domain_name
    - hostmaster@your_domain_name
    - postmaster@your_domain_name
    - webmaster@your_domain_name
    - admin@your_domain_name

Once you have setup an e-mail address with your domain, run `terraform apply` and wait for 2 e-mails. Confirm those e-mails and run `terraform apply` again.

3. You should now have the certificate ready! Wait for the DNS to propagate properly and you should have your domain working shortly. One important thing to keep in mind is that CloudFront only generate certificates for us-east-1 region.
