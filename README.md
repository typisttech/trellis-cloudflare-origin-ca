# Trellis Cloudflare Origin CA

[![Ansible Role](https://img.shields.io/ansible/role/20120.svg)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![GitHub tag](https://img.shields.io/github/tag/TypistTech/trellis-cloudflare-origin-ca.svg)](https://github.com/TypistTech/trellis-cloudflare-origin-ca/tags)
[![license](https://img.shields.io/github/license/TypistTech/trellis-cloudflare-origin-ca.svg)](https://github.com/TypistTech/trellis-cloudflare-origin-ca/blob/master/LICENSE)
[![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://www.typist.tech/donate/sunny/)
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
- [See Also](#see-also)
- [Support!](#support)
  - [Donate via PayPal *](#donate-via-paypal-)
  - [Why don't you hire me?](#why-dont-you-hire-me)
  - [Want to help in other way? Want to be a sponsor?](#want-to-help-in-other-way-want-to-be-a-sponsor)
- [Feedback](#feedback)
- [Author Information](#author-information)
- [Contributing](#contributing)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Requirements

* [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html) 2.3 or later
* [Trellis](https://github.com/roots/trellis)
* [Cloudflare](https://www.cloudflare.com/) account
* Ubuntu 16.04 (Xenial)

## Installation

Add this role to `requirements.yml`:

```yaml
- src: TypistTech.trellis-cloudflare-origin-ca # Case-sensitive!
  version: 0.3.0 # Check for latest version!
```

Run `$ ansible-galaxy install -r requirements.yml` to install this new role.

## Role Variables

```yaml
# group_vars/<environment>/vault.yml
# This file should be encrypted. See: https://roots.io/trellis/docs/vault/
##########################################################################

# Cloudflare Origin CA Key
## Not to confuse with Cloudflare Global API Key
## See: https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#iiobtainyourcertificateapitoken
vault_cloudflare_origin_ca_key: v1.0-xxxxxxxxxxx


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
```

This will generate a *Cloudflare-trusted* certificate for `example.com,hi.example.com,hello.another-example.com`.

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

## See Also

* [Sunny](https://wordpress.org/plugins/sunny/) - Automatically purge CloudFlare cache, including cache everything rules
* [WP Cloudflare Guard](https://wordpress.org/plugins/wp-cloudflare-guard/) - Connecting WordPress with Cloudflare firewall, protect your WordPress site at DNS level. Automatically create firewall rules to block dangerous IPs
* The [Root](https://github.com/roots/trellis/issues/868) of Trellis Cloudflare Origin CA
* The [Origin](https://github.com/roots/trellis/pull/870) of Trellis Cloudflare Origin CA
* [CloudFlare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/)
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

## Author Information

[Trellis Cloudflare Origin CA](https://github.com/TypistTech/trellis-cloudflare-origin-ca) is a [Typist Tech](https://www.typist.tech) project and maintained by [Tang Rufus](https://twitter.com/Tangrufus), freelance developer for [hire](https://www.typist.tech/contact/).

Special thanks to [the Roots team](https://roots.io/about/) whose [Trellis](https://github.com/roots/trellis) make this project possible.

Full list of contributors can be found [here](https://github.com/TypistTech/trellis-cloudflare-origin-ca/graphs/contributors).

## Contributing

Please see [CODE_OF_CONDUCT](./CODE_OF_CONDUCT.md) for details.

## License

[Trellis Cloudflare Origin CA](https://github.com/TypistTech/trellis-cloudflare-origin-ca) is released under the [MIT License](https://opensource.org/licenses/MIT).
