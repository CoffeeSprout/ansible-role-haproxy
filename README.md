coffeesprout.haproxy
=================

Installs HAProxy and configures frontends and backends as required.
Sets up its own self signed cert to bootstrap the proxy https; Default configuration sets up a backend for Let's Encrypt standalone mode.

Recommended to use the haproxy\_cert\_folder and place certificates through other means

Requirements
------------

FreeBSD 11.2+

Role Variables
--------------

    public_ip: "{{ ansible_default_ipv4.address }}"

The public IP that the frontend binds to

    haproxy_default_cert: "{{ haproxy_cert_folder }}/default-combined"
    
The default certificate for HTTPS to bootstrap HAProxy with

    haproxy_cert_folder: "/usr/local/etc/haproxy/certs"

The folder containing all the certificates (HAProxy expects a chain + private key per file)

    haproxy_certificates: []
    
Provide a list of certificates in case you want to be specific about which certificates are served (The cert folder method is more dynamic, all certs in the folder are in scope)

    haproxy_stats_enabled: True

Wether to enable the stats or not

    haproxy_stats_public: True

If the stats are also available to the public

    haproxy_stats_user: admin

You should probably change this to something more secret ;-)

    haproxy_ssl_enabled: True
    
Sometimes you don't need SSL.

    haproxy_alpn_enabled: True

But when you do, you probably want ALPN, H2 etc

    haproxy_hsts_enabled: True

And probably want to announce this using HSTS

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
