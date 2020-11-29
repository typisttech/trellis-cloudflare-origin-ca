<div align="center">

# Trellis Cloudflare Origin CA

</div>

<div align="center">

[![Ansible Role](https://img.shields.io/ansible/role/20120?style=flat-square)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![GitHub tag (latest SemVer)](https://img.shields.io/github/v/tag/TypistTech/trellis-cloudflare-origin-ca?style=flat-square)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![Ansible Role Downloads](https://img.shields.io/ansible/role/d/20120?style=flat-square)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![Ansible Quality Score](https://img.shields.io/ansible/quality/20120?style=flat-square)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![CircleCI](https://img.shields.io/circleci/build/gh/TypistTech/trellis-cloudflare-origin-ca?style=flat-square)](https://circleci.com/gh/TypistTech/trellis-cloudflare-origin-ca)
[![License](https://img.shields.io/github/license/TypistTech/trellis-cloudflare-origin-ca.svg?style=flat-square)](https://github.com/TypistTech/trellis-cloudflare-origin-ca/blob/master/LICENSE)
[![Twitter Follow @TangRufus](https://img.shields.io/twitter/follow/TangRufus?style=flat-square&color=1da1f2&logo=twitter)](https://twitter.com/tangrufus)
[![Hire Typist Tech](https://img.shields.io/badge/Hire-Typist%20Tech-ff69b4.svg?style=flat-square)](https://www.typist.tech/contact/)

</div>

<p align="center">
  <strong>Add Cloudflare Origin CA to Trellis as SSL provider.</strong>
  <br />
  <br />
  Built with ♥ by <a href="https://www.typist.tech/">Typist Tech</a>
</p>

---

**Trellis Cloudflare Origin CA** is an open source project and completely free to use.

However, the amount of effort needed to maintain and develop new features is not sustainable without proper financial backing. If you have the capability, please consider donating using the links below:

<div align="center">

[![GitHub via Sponsor](https://img.shields.io/badge/Sponsor-GitHub-ea4aaa?style=flat-square&logo=github)](https://github.com/sponsors/TangRufus)
[![Sponsor via PayPal](https://img.shields.io/badge/Sponsor-PayPal-blue.svg?style=flat-square&logo=paypal)](https://typist.tech/go/paypal-donate/)
[![More Sponsorship Information](https://img.shields.io/badge/Sponsor-More%20Details-ff69b4?style=flat-square)](https://typist.tech/donate/trellis-cloudflare-origin-ca/)

</div>

---

Add [Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/) to [Trellis](https://github.com/roots/trellis) as a [SSL provider](https://roots.io/trellis/docs/ssl/)

## Why?

Short answer: To keep connection between Cloudflare and your severs private and secure from tampering.

Long answer:
> Cloudflare’s Flexible SSL mode is the default for Cloudflare sites on the Free plan. Flexible SSL mode means that traffic from browsers to Cloudflare will be encrypted, but traffic from Cloudflare to a site's origin server will not be. To take advantage of our [Full and Strict SSL](https://www.cloudflare.com/ssl) mode—which encrypts the connection between Cloudflare and the origin server—it’s necessary to install a certificate on the origin server.
>
> Cloudflare Blog - [Origin Server Connection Security with Universal SSL](https://blog.cloudflare.com/origin-server-connection-security-with-universal-ssl/)

### What are the benefits of Cloudflare Origin CA over Let's Encrypt?

To get certificates from [Let's Encrypt](https://letsencrypt.org/), you have to first disable Cloudflare because Cloudflare hides actual server IPs and make Let's Encrypt challenges fail. Using Cloudflare Origin CA simplifies the troubles.

### What are the benefits of Cloudflare Origin CA over other public certificates?

See [Introducing Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#whataretheincrementalbenefitsoforigincaoverpubliccertificates) on Cloudflare blog.

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
```

---

<p align="center">
  <strong>Typist Tech is ready to build your next awesome WordPress site. <a href="https://typist.tech/contact/">Hire us!</a></strong>
</p>

---

## Requirements

* [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) 2.10 or later
* [Trellis@c86d8a0](https://github.com/roots/trellis/commit/c86d8a042da811e89aa7fdda08159dc86f65be77) or later
* [Cloudflare](https://www.cloudflare.com/) account
* Ubuntu 18.04 (Bionic) or 20.04 (Focal)

## Installation

Add this role to `galaxy.yml`:

```yaml
- src: TypistTech.trellis-cloudflare-origin-ca # Case-sensitive!
  version: 0.8.0 # Check for latest version!
```

Run `$ trellis galaxy install`

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

### `key_type` is deprecated. Please remove it from `example.com`

To avoid misconfiguration, the `key_type` (ECDSA or RSA) and `key_size` (bits) options are deprecated. Since v0.8, this role generates 521-bit ECDSA keys only.

If you had previsously generated CA certificates with other configurations:
1. remove the CA certificates from servers
1. revoke the CA certificates via Cloudflare dashboard
1. re-provision the servers

### `key_size` is deprecated. Please remove it from `example.com`

To avoid misconfiguration, the `key_type` (ECDSA or RSA) and `key_size` (bits) options are deprecated. Since v0.8, this role generates 521-bit ECDSA keys only.

If you had previsously generated CA certificates with other configurations:
1. remove the CA certificates from servers
1. revoke the CA certificates via Cloudflare dashboard
1. re-provision the servers

### Nginx directories not included

Make sure you have [roots/trellis@f2b8107](https://github.com/roots/trellis/commit/f2b81074c83475837e544a8aa5c3e909e760aa8a) or later.

### 400 Bad Request - No required SSL certificate was sent

Symptoms:
* Server returns "400 Bad Request - No required SSL certificate was sent" for all requests
* Nginx logged "client sent no required SSL certificate while reading client request headers, client: [redacted], server:[redacted], request: "GET / HTTP/1.1", host: "[redacted]""
* `ssl_verify_client on;` somewhere in Nginx config files
* Using `client_cert_url` in `wordpress_sites.yml`, i.e: [roots/trellis#869](https://github.com/roots/trellis/pull/869)

Culprit:

Your [Authenticated Origin Pulls](https://support.cloudflare.com/hc/en-us/articles/204899617) configuration is incorrect.

Fact:

This role has nothing to do with Authenticated Origin Pulls or `ssl_verify_client`.

Solution:
1. Read [Introducing Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#whataretheincrementalbenefitsoforigincaoverpubliccertificates)
1. Read [Authenticated Origin Pulls](https://support.cloudflare.com/hc/en-us/articles/204899617)
1. Understand this role is Cloudflare Origin CA
1. Understand Cloudflare Origin CA and Authenticated Origin Pulls are 2 different things
1. Read [#34](https://github.com/TypistTech/trellis-cloudflare-origin-ca/issues/3)
1. Contact Cloudflare support if you still have questions

## FAQ

### Why only 521-bit ECDSA keys allowed?

>I assume you would like to setup [Authenticated Origin Pulls](https://support.cloudflare.com/hc/en-us/articles/204899617-Authenticated-Origin-Pulls) with Cloudflare. I would recommend ECDSA, as elliptic curves provide the same security with less computational overhead.
>
>Find out more about [ECDSA: The digital signature algorithm of a better internet](https://blog.cloudflare.com/ecdsa-the-digital-signature-algorithm-of-a-better-internet/)
>The above article also mentioned that: According to the [ECRYPT II recommendations](http://www.keylength.com/en/3/) on key length, a 256-bit elliptic curve key provides as much protection as a 3,248-bit asymmetric key.Typical RSA keys in website certificates are 2048-bits. So, I think going with 256-bits ECDSA will be a good choice.
>
> --- Cloudflare Support, September 2017

To avoid misconfiguration, the `key_type` (ECDSA or RSA) and `key_size` (bits) options are deprecated. Since v0.8, this role generates 521-bit ECDSA keys only.

If you had previsously generated CA certificates with other configurations:
1. remove the CA certificates from servers
1. revoke the CA certificates via Cloudflare dashboard
1. re-provision the servers

### Why Cloudflare Origin CA key is logged even `cloudflare_origin_ca_no_log` is `true`?

> Note that the use of the `no_log` attribute does not prevent data from being shown when debugging Ansible itself via the `ANSIBLE_DEBUG` environment variable.
>
> --- [Ansible Docs](http://docs.ansible.com/ansible/latest/faq.html#how-do-i-keep-secret-data-in-my-playbook)

### Does Cloudflare Origin CA perfect?

* [Reddit discussion](https://www.reddit.com/r/Monero/comments/73y93c/localmoneroco_uses_cloudflare_which_is_insecure/)
* [Cloudflare, We Have A Problem](http://cryto.net/~joepie91/blog/2016/07/14/cloudflare-we-have-a-problem/)
* [On Cloudflare](https://www.tyil.nl/post/2017/12/17/on-cloudflare/)

### It looks awesome. Where can I find some more goodies like this

- Articles on [Typist Tech's blog](https://typist.tech)
- [Tang Rufus' WordPress plugins](https://profiles.wordpress.org/tangrufus#content-plugins) on wp.org
- More projects on [Typist Tech's GitHub profile](https://github.com/TypistTech)
- Stay tuned on [Typist Tech's newsletter](https://typist.tech/go/newsletter)
- Follow [Tang Rufus' Twitter account](https://twitter.com/TangRufus)
- **Hire [Tang Rufus](https://typist.tech/contact) to build your next awesome site**

### Where can I give 5-star reviews?

Thanks! Glad you like it. It's important to let me knows somebody is using this project. Please consider:

- [tweet](https://twitter.com/intent/tweet?url=https%3A%2F%2Fgithub.com%2FTypistTech%2Ftrellis-cloudflare-origin-ca&via=tangrufus&text=Add%20@Cloudflare%20Origin%20CA%20to%20%23Trellis%20as%20SSL%20provider%20&hashtags=wordpress) something good with mentioning [@TangRufus](https://twitter.com/tangrufus)
- ★ star [the Github repo](https://github.com/TypistTech/trellis-cloudflare-origin-ca)
- [👀 watch](https://github.com/TypistTech/trellis-cloudflare-origin-ca/subscription) the Github repo
- write tutorials and blog posts
- **[hire](https://www.typist.tech/contact/) Typist Tech**

## See Also

* [WP Cloudflare Guard](https://wordpress.org/plugins/wp-cloudflare-guard/) - Connecting WordPress with Cloudflare firewall, protect your WordPress site at DNS level. Automatically create firewall rules to block dangerous IPs
* The [Root](https://github.com/roots/trellis/issues/868) of Trellis Cloudflare Origin CA
* The [Origin](https://github.com/roots/trellis/pull/870) of Trellis Cloudflare Origin CA
* [Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/)
* [Trellis SSL](https://roots.io/trellis/docs/ssl/)
* [Trellis Nginx Includes](https://roots.io/trellis/docs/nginx-includes/)
* [Ansible Vault](https://roots.io/trellis/docs/vault/)

## Running the Tests

Run the tests:

```bash
ansible-playbook -vvv -i 'localhost,' --syntax-check tests/test.yml
ansible-lint -vv .
```

## Feedback

**Please provide feedback!** We want to make this project as useful as possible.
Please [submit an issue](https://github.com/TypistTech/trellis-cloudflare-origin-ca/issues/new) and point out what you do and don't like, or fork the project and [send pull requests](https://github.com/TypistTech/trellis-cloudflare-origin-ca/pulls/).
**No issue is too small.**

## Security Vulnerabilities

If you discover a security vulnerability within this project, please email us at [trellis-cloudflare-origin-ca@typist.tech](mailto:trellis-cloudflare-origin-ca@typist.tech).
All security vulnerabilities will be promptly addressed.

## Credits

[Trellis Cloudflare Origin CA](https://github.com/TypistTech/trellis-cloudflare-origin-ca) is a [Typist Tech](https://typist.tech) project and maintained by [Tang Rufus](https://twitter.com/TangRufus), freelance developer for [hire](https://www.typist.tech/contact/).

Special thanks to [the Roots team](https://roots.io/about/) whose [Trellis](https://github.com/roots/trellis) make this project possible.

Full list of contributors can be found [here](https://github.com/TypistTech/trellis-cloudflare-origin-ca/graphs/contributors).

## License

[Trellis Cloudflare Origin CA](https://github.com/TypistTech/trellis-cloudflare-origin-ca) is released under the [MIT License](https://opensource.org/licenses/MIT).