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
  - [Optiona A: `nginx_wordpress_site_conf`](#optiona-a-nginx_wordpress_site_conf)
  - [Optiona B: `include cloudflare-origin-ca.conf.j2`](#optiona-b-include-cloudflare-origin-caconfj2)
- [Caveats](#caveats)
  - [OCSP stapling should be disabled](#ocsp-stapling-should-be-disabled)
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
  version: 0.2.0 # Check for latest version!
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
      # Stapling must be disabled
      stapling_enabled: false
      # Use this role to generate Cloudflare Origin CA certificate
      provider: cloudflare-origin-ca
```

This will generate a *Cloudflare-trusted* certificate for `example.com,hi.example.com,hello.another-example.com`.

## Hacking Trellis' Playbook

Add this role to `server.yml` **immediately** above `role: wordpress-setup`:

```yaml
roles:
    # Some other Trellis roles ...
    - { role: TypistTech.trellis-cloudflare-origin-ca, tags: [cloudflare-origin-ca], when: sites_using_cloudflare_origin_ca | count }
    - { role: wordpress-setup, tags: [wordpress, wordpress-setup, letsencrypt, cloudflare-origin-ca] }
    # Some other Trellis roles ...
```

Note: `role: wordpress-setup` is tagged with `cloudflare-origin-ca`.

## Nginx Includes

### Optiona A: `nginx_wordpress_site_conf`

Define `nginx_wordpress_site_conf` in your `group_vars/all/main.yml` to use this role's Nginx site template:

```yaml
nginx_wordpress_site_conf: vendor/roles/TypistTech.trellis-cloudflare-origin-ca/templates/wordpress-site.conf.child
```

### Optiona B: `include cloudflare-origin-ca.conf.j2`

If you already using a [child template](https://roots.io/trellis/docs/nginx-includes/), add this line to every server block that use Cloudflare Origin CA:

```
{% include 'vendor/roles/TypistTech.trellis-cloudflare-origin-ca/templates/cloudflare-origin-ca.conf.j2' %}
```

## Caveats

### OCSP stapling should be disabled

> ... you're trying to staple OCSP responses with Origin CA. Right now OCSP is not supported with Origin CA, so you should remove the ssl_staping directive for the host that you're using the Origin CA cert on...
> --- Cloudflare Support

## See Also

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
