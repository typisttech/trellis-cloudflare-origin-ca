# Trellis Cloudflare Origin CA

[![Ansible Role](https://img.shields.io/ansible/role/20120.svg)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![Build Status](https://travis-ci.org/TypistTech/trellis-cloudflare-origin-ca.svg?branch=master)](https://travis-ci.org/TypistTech/trellis-cloudflare-origin-ca)
[![GitHub tag](https://img.shields.io/github/tag/TypistTech/trellis-cloudflare-origin-ca.svg)](https://github.com/TypistTech/trellis-cloudflare-origin-ca/tags)
[![license](https://img.shields.io/github/license/TypistTech/trellis-cloudflare-origin-ca.svg)](https://github.com/TypistTech/trellis-cloudflare-origin-ca/blob/master/LICENSE)
[![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://www.typist.tech/donate/trellis-cloudflare-origin-ca/)
[![Hire Typist Tech](https://img.shields.io/badge/Hire-Typist%20Tech-ff69b4.svg)](https://www.typist.tech/contact/)

Add [Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/) to [Trellis](https://github.com/roots/trellis) as a [SSL provider](https://roots.io/trellis/docs/ssl/)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Requirements](#requirements)
- [Installation](#installation)
- [Role Variables](#role-variables)
- [Hacking Trellis' Playbook](#hacking-trellis-playbook)
- [Nginx Includes](#nginx-includes)
- [Common Errors](#common-errors)
  - [No site is using Cloudflare Origin CA](#no-site-is-using-cloudflare-origin-ca)
  - [`vault_cloudflare_origin_ca_key` is not defined](#vault_cloudflare_origin_ca_key-is-not-defined)
  - [`example.com` is using Cloudflare Origin CA but OCSP stapling is enabled](#examplecom-is-using-cloudflare-origin-ca-but-ocsp-stapling-is-enabled)
  - [Nginx directories not included](#nginx-directories-not-included)
  - [400 Bad Request - No required SSL certificate was sent](#400-bad-request---no-required-ssl-certificate-was-sent)
- [FAQ](#faq)
  - [Why use Cloudflare Origin CA?](#why-use-cloudflare-origin-ca)
  - [What are the benefits of Cloudflare Origin CA over Let's Encrypt?](#what-are-the-benefits-of-cloudflare-origin-ca-over-lets-encrypt)
  - [What are the benefits of Cloudflare Origin CA over other public certificates?](#what-are-the-benefits-of-cloudflare-origin-ca-over-other-public-certificates)
  - [Why use 256-bit ECDSA key as default?](#why-use-256-bit-ecdsa-key-as-default)
  - [Why Cloudflare Origin CA key is logged even `cloudflare_origin_ca_no_log` is `true`?](#why-cloudflare-origin-ca-key-is-logged-even-cloudflare_origin_ca_no_log-is-true)
  - [Does Cloudflare Origin CA perfect?](#does-cloudflare-origin-ca-perfect)
- [See Also](#see-also)
- [Support!](#support)
  - [Donate via PayPal *](#donate-via-paypal-)
  - [Why don't you hire me?](#why-dont-you-hire-me)
  - [Want to help in other way? Want to be a sponsor?](#want-to-help-in-other-way-want-to-be-a-sponsor)
- [Feedback](#feedback)
- [Change log](#change-log)
- [Author Information](#author-information)
- [Contributing](#contributing)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Requirements

* [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html) 2.3 or later
* [Trellis@f2b8107](https://github.com/roots/trellis/commit/f2b81074c83475837e544a8aa5c3e909e760aa8a) or later
* [Cloudflare](https://www.cloudflare.com/) account
* Ubuntu 16.04 (Xenial)

## Installation

Add this role to `requirements.yml`:

```yaml
- src: TypistTech.trellis-cloudflare-origin-ca # Case-sensitive!
  version: 0.6.0 # Check for latest version!
```

Run `$ ansible-galaxy install -r requirements.yml` to install this new role.

## Role Variables

```yaml
# group_vars/<environment>/vault.yml
# This file should be encrypted. See: https://roots.io/trellis/docs/vault/
##########################################################################

# Cloudflare Origin CA Key
# Not to confuse with Cloudflare Global API Key
# See: https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#iiobtainyourcertificateapitoken
vault_cloudflare_origin_ca_key: v1.0-xxxxxxxxxxx

# group_vars/<environment>/main.yml
###################################

# Indicates the desired package state.
# `latest` ensures that the latest version is installed.
# `present` does not update if already installed.
# Choices: present|latest
# Default: latest
cfca_package_state: present

# Whether to hide results of sensitive tasks which
# may include Cloudflare Origin CA Key in plain text.
# Choices: true|false
# Default: false
cloudflare_origin_ca_no_log: true

# group_vars/<environment>/wordpress_sites.yml
##############################################

wordpress_sites:
  example.com:
    # Your Cloudflare account must own all these domains
    site_hosts:
      - canonical: example.com
        redirects:
          - hi.example.com
          - hello.another-example.com
    ssl:
      # SSL must be enabled
      enabled: true
      # OCSP stapling must be disabled
      stapling_enabled: false
      # Use this role to generate Cloudflare Origin CA certificate
      provider: cloudflare-origin-ca
    # The followings are optional
    cloudflare_origin_ca:
      # Number of days for which the issued cert will be valid. Acceptable options are: 7, 30, 90, 365 (1y), 730 (2y), 1095 (3y), 5475 (15y).
      # Default: 5475
      days: 7
      # List of fully-qualified domain names to include on the certificate as Subject Alternative Names.
      # Default: All canonical and redirect domains
      # In the above example: example.com, hi.example.com, hello.another-example.com
      hostnames:
        - example.com
        - '*.example.com'
        - '*.another-example.com'
      # Key size in bits to use for the generated key pair (Acceptable sizes: rsa: 2048|3072|4096, ecdsa: 256|384|521)
      # Default: 256
      key_size: 3072
      # Type of key pair to generate, either RSA or ECDSA. (rsa|ecdsa)
      # Default: ecdsa
      key_type: rsa
```

## Hacking Trellis' Playbook

Add this role to `server.yml` **immediately after** `role: wordpress-setup`:

```yaml
roles:
    # Some other Trellis roles ...
    - { role: wordpress-setup, tags: [wordpress, wordpress-setup, letsencrypt, cloudflare-origin-ca] }
    - { role: TypistTech.trellis-cloudflare-origin-ca, tags: [cloudflare-origin-ca, wordpress-setup], when: sites_using_cloudflare_origin_ca | count }
    # Some other Trellis roles ...
```

Note: `role: wordpress-setup` is tagged with `cloudflare-origin-ca`.

## Nginx Includes

This role templates Nginx SSL directives out to `{{ nginx_path }}/includes.d/{{ item.key }}/cloudflare-origin-ca.conf`. Trellis includes this file [here](https://github.com/roots/trellis/blob/4cd1be12a8cfacf78af3a9a1302bea153f80e459/roles/wordpress-setup/templates/wordpress-site.conf.j2#L106) and [here](https://github.com/roots/trellis/pull/879/files) by default, no action needed.

If you using [Nginx child templates](https://roots.io/trellis/docs/nginx-includes/#child-templates), add this line into your server blocks:
```
include includes.d/{{ item.key }}/cloudflare-origin-ca.conf;
```

## Common Errors

### No site is using Cloudflare Origin CA

Obviously, you should not run this role when you don't use Cloudflare Origin CA.

### `vault_cloudflare_origin_ca_key` is not defined

Encrypt your Cloudflare Origin CA Key in `group_vars/<environment>/vault.yml`. See [role variables](#role-variables).

### `example.com` is using Cloudflare Origin CA but OCSP stapling is enabled

> ... you're trying to staple OCSP responses with Origin CA. Right now OCSP is not supported with Origin CA, so you should remove the ssl_staping directive for the host that you're using the Origin CA cert on...
>
> --- Cloudflare Support

Cloudflare Origin CA doesn't support OCSP stapling. Disable OCSP stapling for all sites using Cloudflare Origin CA. See [role variables](#role-variables).

### Nginx directories not included

Make sure you have [roots/trellis@f2b8107](https://github.com/roots/trellis/commit/f2b81074c83475837e544a8aa5c3e909e760aa8a) or later.

### 400 Bad Request - No required SSL certificate was sent

Symptoms:
- Server returns "400 Bad Request - No required SSL certificate was sent" for all requests
- Nginx logged "client sent no required SSL certificate while reading client request headers, client: [redacted], server:[redacted], request: "GET / HTTP/1.1", host: "[redacted]""
- `ssl_verify_client on;` somewhere in Nginx config files
- Using `client_cert_url` in `wordpress_sites.yml`, i.e: [roots/trellis#869](https://github.com/roots/trellis/pull/869)

Culprit:
Your ["Authenticated Origin Pulls"](https://support.cloudflare.com/hc/en-us/articles/204899617) configuration is incorrect.

Fact:
This role has nothing to do with authenticated origin pulls or `ssl_verify_client`.

Solution:
1. Read [Introducing Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#whataretheincrementalbenefitsoforigincaoverpubliccertificates)
1. Read [Authenticated Origin Pulls](https://support.cloudflare.com/hc/en-us/articles/204899617)
1. Understand this role is Cloudflare Origin CA
1. Understand Cloudflare Origin CA and Authenticated Origin Pulls are 2 different things
1. Read [#34](https://github.com/TypistTech/trellis-cloudflare-origin-ca/issues/3
1. Contact Cloudflare support if you still have questions

## FAQ

### Why use Cloudflare Origin CA?

Short answer: To keep connection between Cloudflare and your severs private and secure from tampering.

Long answer:
> Cloudflare’s Flexible SSL mode is the default for Cloudflare sites on the Free plan. Flexible SSL mode means that traffic from browsers to Cloudflare will be encrypted, but traffic from Cloudflare to a site's origin server will not be. To take advantage of our [Full and Strict SSL](https://www.cloudflare.com/ssl) mode—which encrypts the connection between Cloudflare and the origin server—it’s necessary to install a certificate on the origin server.
>
> Cloudflare Blog - [Origin Server Connection Security with Universal SS ](https://blog.cloudflare.com/origin-server-connection-security-with-universal-ssl/)

### What are the benefits of Cloudflare Origin CA over Let's Encrypt?

To get certificates from [Let's Encrypt](https://letsencrypt.org/), you have to first disable Cloudflare because Cloudflare hides actual server IPs and make Let's Encrypt challenges fail. Using Cloudflare Origin CA simplify the troubles.

### What are the benefits of Cloudflare Origin CA over other public certificates?

See [Introducing Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#whataretheincrementalbenefitsoforigincaoverpubliccertificates) on Cloudflare blog.

### Why use 256-bit ECDSA key as default?

>I assume you would like to setup [Authenticated Origin Pulls](https://support.cloudflare.com/hc/en-us/articles/204899617-Authenticated-Origin-Pulls) with Cloudflare. I would recommend ECDSA, as elliptic curves provide the same security with less computational overhead.
>
>Find out more about [ECDSA: The digital signature algorithm of a better internet](https://blog.cloudflare.com/ecdsa-the-digital-signature-algorithm-of-a-better-internet/)
>The above article also mentioned that: According to the [ECRYPT II recommendations](http://www.keylength.com/en/3/) on key length, a 256-bit elliptic curve key provides as much protection as a 3,248-bit asymmetric key.Typical RSA keys in website certificates are 2048-bits. So, I think going with 256-bits ECDSA will be a good choice.
>
> --- Cloudflare Support

If you insist to use RSA keys, make sure you set `key_size` to at least `2048`.

### Why Cloudflare Origin CA key is logged even `cloudflare_origin_ca_no_log` is `true`?

> Note that the use of the `no_log` attribute does not prevent data from being shown when debugging Ansible itself via the `ANSIBLE_DEBUG` environment variable.
>
> [Ansible Docs](http://docs.ansible.com/ansible/latest/faq.html#how-do-i-keep-secret-data-in-my-playbook)

### Does Cloudflare Origin CA perfect?

* [Reddit discussion](https://www.reddit.com/r/Monero/comments/73y93c/localmoneroco_uses_cloudflare_which_is_insecure/)
* [Cloudflare, We Have A Problem](http://cryto.net/~joepie91/blog/2016/07/14/cloudflare-we-have-a-problem/)
* [On Cloudflare](https://www.tyil.nl/articles/on-cloudflare/)

## See Also

* [Sunny](https://wordpress.org/plugins/sunny/) - Automatically purge Cloudflare cache, including cache everything rules
* [WP Cloudflare Guard](https://wordpress.org/plugins/wp-cloudflare-guard/) - Connecting WordPress with Cloudflare firewall, protect your WordPress site at DNS level. Automatically create firewall rules to block dangerous IPs
* The [Root](https://github.com/roots/trellis/issues/868) of Trellis Cloudflare Origin CA
* The [Origin](https://github.com/roots/trellis/pull/870) of Trellis Cloudflare Origin CA
* [Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/)
* [Trellis SSL](https://roots.io/trellis/docs/ssl/)
* [Trellis Nginx Includes](https://roots.io/trellis/docs/nginx-includes/)
* [Ansible Vault](https://roots.io/trellis/docs/vault/)

## Support!

### Donate via PayPal [![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://www.typist.tech/donate/trellis-cloudflare-origin-ca/)

Love Trellis Cloudflare Origin CA? Help me maintain it, a [donation here](https://www.typist.tech/donate/trellis-cloudflare-origin-ca/) can help with it.

### Why don't you hire me?

Ready to take freelance WordPress jobs. Contact me via the contact form [here](https://www.typist.tech/contact/) or, via email [info@typist.tech](mailto:info@typist.tech)

### Want to help in other way? Want to be a sponsor?

Contact: [Tang Rufus](mailto:tangrufus@gmail.com)

## Feedback

**Please provide feedback!** We want to make this library useful in as many projects as possible.
Please submit an [issue](https://github.com/TypistTech/trellis-cloudflare-origin-ca/issues/new) and point out what you do and don't like, or fork the project and make suggestions.
**No issue is too small.**

## Change log

Please see [CHANGELOG](./CHANGELOG.md) for more information on what has changed recently.

## Author Information

[Trellis Cloudflare Origin CA](https://github.com/TypistTech/trellis-cloudflare-origin-ca) is a [Typist Tech](https://www.typist.tech) project and maintained by [Tang Rufus](https://twitter.com/Tangrufus), freelance developer for [hire](https://www.typist.tech/contact/).

Special thanks to [the Roots team](https://roots.io/about/) whose [Trellis](https://github.com/roots/trellis) make this project possible.

Full list of contributors can be found [here](https://github.com/TypistTech/trellis-cloudflare-origin-ca/graphs/contributors).

## Contributing

Please see [CODE_OF_CONDUCT](./CODE_OF_CONDUCT.md) for details.

## License

[Trellis Cloudflare Origin CA](https://github.com/TypistTech/trellis-cloudflare-origin-ca) is released under the [MIT License](https://opensource.org/licenses/MIT).
