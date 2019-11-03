# acme-client-portable

This is an unoffical port of acme-client(1) from OpenBSDs releases.

# acme-client(1)

```
ACME-CLIENT(1)              General Commands Manual             ACME-CLIENT(1)

NAME
     acme-client  ACME client

SYNOPSIS
     acme-client [-Fnrv] [-f configfile] domain

DESCRIPTION
     acme-client is an Automatic Certificate Management Environment (ACME)
     client: it looks in its configuration for a domain section corresponding
     to the domain given as command line argument and uses that configuration
     to retrieve an X.509 certificate which can be used to provide domain name
     validation (i.e. prove that the domain is who it says it is).  The
     certificates are typically used to provide HTTPS for web servers, but can
     be used in any situation where domain name validation is required (such
     as mail servers).

     If the certificate already exists and is less than 30 days from expiry,
     acme-client attempts to renew the certificate.

     In order to prove that the client has access to the domain, a challenge
     is issued by the signing authority.  acme-client implements the http-01
     challenge type, where a file is created within a directory accessible by
     a locally run web server.  The default challenge directory /var/www/acme
     can be served by httpd(8) with this location block, which will properly
     map response challenges:

           location "/.well-known/acme-challenge/*" {
                   root "/acme"
                   request strip 2
           }

     The options are as follows:

     -F      Force certificate renewal, even if it's too soon.

     -f configfile
             Specify an alternative configuration file.

     -n      No operation: check and print configuration.

     -r      Revoke the X.509 certificate.

     -v      Verbose operation.  Specify twice to also trace communication and
             data transfers.

     domain  The domain name.

FILES
     /etc/acme              Private keys for acme-client.
     /etc/acme-client.conf  Default configuration.
     /var/www/acme          Default challengedir.

EXIT STATUS
     acme-client returns 0 if certificates were changed (revoked or updated),
     1 on failure, or 2 if the certificates didn't change (up to date).

EXAMPLES
     Example configuration files for acme-client and httpd(8) are provided in
     /etc/examples/acme-client.conf and /etc/examples/httpd.conf.

     To generate a certificate for example.com and use it to provide HTTPS,
     create acme-client.conf and httpd.conf and run:

           # acme-client -v example.com && rcctl reload httpd

     A cron(8) job can renew the certificate as necessary.  On renewal,
     httpd(8) is reloaded:

           0       *       *       *       *       sleep $((RANDOM \% 2048)) && \
                   acme-client example.com && rcctl reload httpd

SEE ALSO
     openssl(1), acme-client.conf(5), httpd.conf(5)

STANDARDS
     R. Barnes, J. Hoffman-Andrews, D. McCarney, and J. Kasten, Automatic
     Certificate Management Environment (ACME), RFC 8555, March 2019.

HISTORY
     The acme-client utility first appeared in OpenBSD 6.1.

AUTHORS
     The acme-client utility was written by Kristaps Dzonsons
     <kristaps@bsd.lv>.

Void Linux                       June 15, 2019                      Void Linux
```

# acme-client.conf(5)

```
ACME-CLIENT.CONF(5)           File Formats Manual          ACME-CLIENT.CONF(5)

NAME
     acme-client.conf  acme-client configuration file

DESCRIPTION
     The acme-client.conf file is divided into the following main sections:

     Macros
           User-defined variables may be defined and used later, simplifying
           the configuration file.

     Authorities
           Certificate authorities (CAs) that can be contacted via ACME.

     Domains
           Certificate specifications.

     Additional configuration files can be included with the include keyword,
     for example:

           include "/etc/acme-client.sub.conf"

     The current line can be extended over multiple lines using a backslash
     (\).  Comments can be put anywhere in the file using a hash mark (#),
     and extend to the end of the current line.  Care should be taken when
     commenting out multi-line text: the comment is effective until the end of
     the entire block.

     Argument names not beginning with a letter, digit, underscore, or '/'
     must be quoted.

MACROS
     Macros can be defined that will later be expanded in context.  Macro
     names must start with a letter, digit, or underscore, and may contain any
     of those characters.  Macro names may not be reserved words.  Macros are
     not expanded inside quotes.

     For example:

           api_url="https://acme-v02.api.letsencrypt.org/directory"
           authority letsencrypt {
                   api url $api_url
                   account key "/etc/acme/letsencrypt-privkey.pem"
           }

AUTHORITIES
     The configured certificate authorities.

     Each authority section starts with a declaration of the name identifying
     a certificate authority.

     authority name {...}
             The name is a string used to reference this certificate
             authority.

     It is followed by a block of options enclosed in curly brackets:

     account key file [keytype]
             Specify a file used to identify the user of this certificate
             authority.  keytype can be rsa or ecdsa.  It defaults to rsa.

     api url url
             Specify the url under which the ACME API is reachable.

DOMAINS
     The certificates to be obtained through ACME.

     domain name {...}
             Each domain section begins with the domain keyword followed by
             the name to be used as the common name component of the subject
             of the X.509 certificate.

     It is followed by a block of options enclosed in curly brackets:

     alternative names {...}
             Specify a list of alternative names for which the certificate
             will be valid.  The common name is included automatically if this
             option is present, but there is no automatic conversion/inclusion
             between "www." and plain domain name forms.

     domain key file [keytype]
             The private key file for which the certificate will be obtained.
             keytype can be rsa or ecdsa.  It defaults to rsa.

     domain certificate file
             The filename of the certificate that will be issued.  This is
             optional if domain full chain certificate is specified.

     domain chain certificate file
             The filename in which to store the certificate chain that will be
             returned by the certificate authority.  It needs to be in the
             same directory as the domain certificate (or in a subdirectory)
             and can be specified as a relative or absolute path.  This
             setting is optional.

     domain full chain certificate file
             The filename in which to store the full certificate chain that
             will be returned by the certificate authority.  It needs to be in
             the same directory as the domain certificate (or in a
             subdirectory) and can be specified as a relative or absolute
             path.  This is a combination of the domain certificate and the
             domain chain certificate in one file, and is required by most
             browsers.  This is optional if domain certificate is specified.

     sign with authority
             The certificate authority (as declared above in the AUTHORITIES
             section) to use.  If this setting is absent, the first authority
             specified is used.

     challengedir path
             The directory in which the challenge file will be stored.  If it
             is not specified, a default of /var/www/acme will be used.

FILES
     /etc/acme-client.conf  acme-client(1) configuration file

SEE ALSO
     acme-client(1)

HISTORY
     The acme-client.conf file format first appeared in OpenBSD 6.1.

Void Linux                       July 4, 2019                       Void Linux
```
