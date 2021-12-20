===========================
 Ansible Role: ldap_server
===========================

This role deploys an OpenLDAP server with a pre-generated TLS certificate.

----------------
 Role Variables
----------------

Key role variables are documented with their default values below. See ``defaults/main.yml`` for a full list.

::

    ldap_server_dn: "dc=example,dc=com"
    ldap_server_dc: "example"

Sets the base domain for which the server will be responsible.

::

    ldap_server_password: ""

The administrative password for the server

::

    ldap_server_cert_path: "../keys/ldap_server/cert.pem"
    ldap_server_priv_path: "../keys/ldap_server/priv.pem"

Where on the ansible server to find the private key and certificate which the server will use.

::

    ldap_server_schemas:
        - /etc/openldap/schema/cosine.ldif
        - /etc/openldap/schema/nis.ldif
        - /etc/openldap/schema/inetorgperson.ldif

Which schemas modules to load into the server.

:: 
    
    ldap_server_sasl_enabled: False
    ldap_server_sasl_servers: ""
    ldap_server_sasl_search_base: ""
    ldap_server_sasl_filter: "uid=%U"
    ldap_server_sasl_bind_dn: ""
    ldap_server_sasl_password: ""

If desired, SASL to a second LDAP or AD server can be configured using these settings.

------------------
 Example Playbook
------------------

::

    ---
    - hosts: localhost
      vars:
        ldap_server_dn: "dc=home,dc=example,dc=com"
        ldap_server_dc: "example"
        ldap_server_password: "{{ lookup('file', 'keys/ldap_server/password.txt') }}"
      roles:
        - ldap_server

------------------------------
 Generating a new certificate
------------------------------

The role provides a mechanism for generating a new self-signed certificate 
using the existing private key should the existing certificate expire or
otherwise need to be replaced. To use this feature, first define these 
additional variables for the server host:

::

    ldap_server_subj: ""
    ldap_server_renew_period: 365

These settings set the subject and valid lifetime of the generated certificate.

Then, run a playbook including this role with the tag ``ldap_server_cert_renew``. 
This will generate a new certificate on the LDAP server host, copy it back onto
the ansible server (replacing the current ``ldap_server_cert_path`` file), and 
then restart the SLAPD service so that it starts using the new certificate.

--------------
 SASL Support
--------------

LDAP allows delegating authentication of a user to a different LDAP or AD server
via SASL. This is done by setting the ``userPassword`` value on a user entry to
``{SASL}<uid>``, where ``<uid>`` is the user's identity on the remote server.
When a user tries to authenticate, the SASLAuthd service on the LDAP server will
resend the password used along with the new username ``<uid>`` to the upstream
authentication server, then passes through the success or failure result.

To enable and utilize SASL, define the following additional host/group variables:

::

    ldap_server_sasl_enabled: False
    ldap_server_sasl_servers: ""
    ldap_server_sasl_search_base: ""
    ldap_server_sasl_filter: "uid=%U"
    ldap_server_sasl_bind_dn: ""
    ldap_server_sasl_password: ""
