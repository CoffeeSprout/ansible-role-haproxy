coffeesprout.haproxy
=================

Installs HAProxy and configures frontends and backends as required.
Sets up its own self signed cert to bootstrap the proxy https; Default configuration sets up a backend for Let's Encrypt standalone mode.

Recommended to use the haproxy\_cert\_folder and place certificates through other means

Now also supports HAProxy map_dom for binding domains and backends in a more flexible and less verbose way.

Requirements
------------

FreeBSD 11.2+ ; Debian 11+

Role Variables
--------------

Here are the variables available to configure the role:

- `public_ip`: The public IP that the frontend binds to.
- `haproxy_default_cert`: The default certificate for HTTPS to bootstrap HAProxy with.
- `haproxy_cert_folder`: The folder containing all the certificates (HAProxy expects a chain + private key per file).
- `haproxy_certificates`: A list of certificates in case you want to be specific about which certificates are served (the cert folder method is more dynamic, all certs in the folder are in scope).
- `haproxy_stats_enabled`: Whether to enable the stats or not.
- `haproxy_stats_public`: If the stats are also available to the public.
- `haproxy_stats_user`: The username to access HAProxy stats page.
- `haproxy_ssl_enabled`: Whether SSL should be enabled.
- `haproxy_alpn_enabled`: Whether to enable ALPN, H2 etc.
- `haproxy_hsts_enabled`: Whether to enable HSTS.


The `sites` variable is the most critical part of the role. It defines the sites that we want to service in HAProxy. Here's an example configuration:

    sites: []
    #  - name: coffeesprout
    #    domains:
    #    - www.coffeesprout.nl
    #    - coffeesprout.nl
    #    servers:
    #    - id: server1
    #      hostname: 192.168.2.175
    #    - id: server2
    #      hostname: 192.168.2.176
    #      port: 9999
    #    extras:
    #    - "option httpchk HEAD /"

Defines the sites that we want to service in HAProxy. The name is administrative and determines the backend name; The domains are a list of domains we listen for. The list of servers includes a list of backend servers and their ip's / hostname.

You may also want to add basic auth on a per website basis. First you define the userlist(s):

    haproxy_userlists: []

We expect a list of userlists, for example:

    haproxy_userlists:
    - name: admin
      members:
      - user: barry
        password: notverysecure
      - user: manager
        password: kingofthecastle
        
Add the realm to the site instance using:

      http_auth: admin

Selects the userlist by name

      remove_auth_header: True

Some applications don't like receiving the Authentication header, in that case we can filter it with above.

       haproxy_multi_config: False

HAProxy by default only loads from a single config file; It will also support loading all files in a directory. Setting this to True makes it load the os specific haproxy_config_folder instead, allowing you to split your configs



Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: lbs
      become: True
      vars:
        letsencrypt_host: "10.0.0.1"
      handlers:
        - name: Reload syslog
          service:
            name: syslogd
            state: reloaded
      roles:
      - role: coffeesprout.pf
        pf_skip:
        - vtnet1
        - vtnet2
        pf_tcp_pass_in:
        - 80
        - 443
      - role: coffeesprout.haproxy
        sites:
        - name: site-prd
          domains:
          - site.example.com
          servers:
          - id: application00
            hostname: 10.0.1.1
          - id: application01
            hostname: 10.0.1.2
        - name: site-acc
          domains:
          - site-acc.example.com
          servers:
          - id: accept00
            hostname: 10.0.0.1
    



License
-------

BSD

Author Information
------------------

Just reach out via Github
