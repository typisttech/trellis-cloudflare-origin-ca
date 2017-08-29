# Trellis Cloudflare Origin CA

[![Ansible Role](https://img.shields.io/ansible/role/20120.svg)](https://galaxy.ansible.com/TypistTech/trellis-cloudflare-origin-ca/)
[![license](https://img.shields.io/github/license/TypistTech/trellis-cloudflare-origin-ca.svg)](https://github.com/TypistTech/trellis-cloudflare-origin-ca/blob/master/LICENSE)
[![Donate via PayPal](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://www.typist.tech/donate/sunny/)
[![Hire Typist Tech](https://img.shields.io/badge/Hire-Typist%20Tech-ff69b4.svg)](https://www.typist.tech/contact/)

Add [Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/) to [Trellis](https://github.com/roots/trellis) as a [SSL provider](https://roots.io/trellis/docs/ssl/)

## Requirements

* [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html) 2.3 or later
* Trellis [4459ab5](https://github.com/roots/trellis/commit/4459ab5b9eb7f7cd235debb62eab23ba18820b72) or later
* [Cloudflare](https://www.cloudflare.com/) account
* Ubuntu 16.04 (Xenial)
* Read about [CloudFlare Origin CA] (https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#iiobtainyourcertificateapitoken)
* Read about [Trellis SSL](https://roots.io/trellis/docs/ssl/)
* Read about [Trellis Nginx Includes](https://roots.io/trellis/docs/nginx-includes/)
* Read about [Ansible Vault](https://roots.io/trellis/docs/vault/)

## Installation

Add this role to `requirements.yml`:

```yaml
- src: TypistTech.trellis-cloudflare-origin-ca # Case-sensitive!
```

Run `ansible-galaxy install -r requirements.yml` to install this new role.

## Role Variables

### `vault_cloudflare_origin_ca_key`

Define Cloudflare `Origin CA Key` as `vault_cloudflare_origin_ca_key` [secretly](https://roots.io/trellis/docs/vault/) in `group_vars/<environment>/vault.yml`:

```yaml
vault_cloudflare_origin_ca_key: v1.0-xxxxxxxxxxx
```

Note:
* `Origin CA Key` and `Global API Key` are different.
* [How to obtain your Cloudflare `Origin CA Key`?](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#iiobtainyourcertificateapitoken)

### `nginx_wordpress_site_conf`

Define `nginx_wordpress_site_conf` in your `group_vars/all/main.yml` to use this role's nginx site template::

```yaml
nginx_wordpress_site_conf: vendor/roles/TypistTech.trellis-cloudflare-origin-ca/templates/wordpress-site.conf.child
```

### `provider: cloudflare-origin-ca`
Set `provider: cloudflare-origin-ca` in `group_vars/<environment>/wordpress_sites.yml`:

```yaml
wordpress_sites:
  example.com:
    site_hosts:
      - canonical: example.com
        redirects:
          - hi.example.com
          - bye.example.com
    ssl:
      enabled: true # Must be enabled
      provider: cloudflare-origin-ca # Add this line
```

This will generate a *Cloudflare-trusted* certificate for `example.com,hi.example.com,bye.example.com`.

## Hacking Trellis' Playbook

Add this role to `server.yml` **immediately** above `role: wordpress-setup`:

```yaml
roles:
    # Some other Trellis roles ...
    - { role: TypistTech.trellis-cloudflare-origin-ca, tags: [cloudflare-origin-ca] } # Case-sensitive!
    - { role: wordpress-setup, tags: [wordpress, wordpress-setup] }
    # Some other Trellis roles ...
```

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
