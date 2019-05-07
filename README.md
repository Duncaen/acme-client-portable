# acme-client-portable

This is an unoffical port of acme-client(1) from OpenBSDs releases.

# acme-client(1)

```
ACME-CLIENT(1)              General Commands Manual             ACME-CLIENT(1)

NAME
     acme-client  ACME client

SYNOPSIS
     acme-client [-ADFnrv] [-f configfile] domain

DESCRIPTION
     The acme-client utility is an Automatic Certificate Management
     Environment (ACME) client.

     The options are as follows:

     -A      Create a new RSA account key if one does not already exist.

     -D      Create a new RSA domain key if one does not already exist.

     -F      Force updating the certificate signature even if it's too soon.

     -f configfile
             Specify an alternative configuration file.

     -n      No operation: check and print configuration.

     -r      Revoke the X.509 certificate.

     -v      Verbose operation.  Specify twice to also trace communication and
             data transfers.

     domain  The domain name.

     acme-client looks in its configuration for a domain section corresponding
     to the domain given as command line argument.  It then uses that
     configuration to retrieve an X.509 certificate.  If the certificate
     already exists and is less than 30 days from expiry, acme-client will
     attempt to refresh the signature.  Before a certificate can be requested,
     an account key needs to be created using the -A argument.  The first time
     a certificate is requested, the RSA key needs to be created with -D.

     Challenges are used to verify that the submitter has access to the
     registered domains.  acme-client only implements the http-01 challenge
     type, where a file is created within a directory accessible by a locally-
     run web server.  The default challenge directory /var/www/acme can be
     served by httpd(8) with this location block, which will properly map
     response challenges:

           location "/.well-known/acme-challenge/*" {
                   root "/acme"
                   request strip 2
           }

FILES
     /etc/acme-client.conf  Default configuration.
     /var/www/acme          Default challengedir.

EXIT STATUS
     acme-client returns 1 on failure, 2 if the certificates didn't change (up
     to date), or 0 if certificates were changed (revoked or updated).

EXAMPLES
     To initialize a new account and Domain key:

           # acme-client -vAD example.com

     To create and submit a new key for a single domain, assuming that the web
     server has already been configured to map the challenge directory as
     above:

           # acme-client -vD example.com

     A daily cron(8) job can renew the certificate:

           acme-client example.com && rcctl reload httpd

SEE ALSO
     openssl(1), acme-client.conf(5), httpd.conf(5)

STANDARDS
     Automatic Certificate Management Environment (ACME),
     https://tools.ietf.org/html/draft-ietf-acme-acme-03.

HISTORY
     The acme-client utility first appeared in OpenBSD 6.1.

AUTHORS
     The acme-client utility was written by Kristaps Dzonsons
     <kristaps@bsd.lv>.

BUGS
     The challenge and certificate processes currently retain their (root)
     privileges.

     For the time being, acme-client only supports RSA as an account key
     format.

Void Linux                      August 2, 2018                      Void Linux
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

           le="letsencrypt"
           domain example.com {
                   sign with $le
           }

AUTHORITIES
     The configured certificate authorities.

     Each authority section starts with a declaration of the name identifying
     a certificate authority.

     authority name {...}
             The name is a string used to reference this certificate
             authority.

     It is followed by a block of options enclosed in curly brackets:

     account key file
             Specify a file used to identify the user of this certificate
             authority.

     api url url
             Specify the url under which the ACME API is reachable.

     An example authority block:

           authority letsencrypt {
                   api url "https://acme-v01.api.letsencrypt.org/directory"
                   account key "/etc/ssl/private/my-acme.key"
           }

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

     domain key file
             The private key file for which the certificate will be obtained.

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

     An example domain declaration looks like this:

           domain www.example.com {
                   alternative names { example.com secure.example.com }
                   domain key "/etc/ssl/private/www.example.com.key"
                   domain certificate "/etc/ssl/www.example.com.crt"
                   domain full chain certificate "/etc/ssl/www.example.com.fullchain.pem"
                   sign with letsencrypt
                   challengedir "/var/www/acme"
           }

     An httpd.conf(5) server declaration to use that certificate looks like
     this:

           server "www.example.com" {
                   alias "example.com"
                   alias "secure.example.com"
                   listen on $ext_addr port 80
                   listen on $ext_addr tls port 443
                   tls certificate "/etc/ssl/www.example.com.fullchain.pem"
                   tls key "/etc/ssl/private/www.example.com.key"
                   location "/.well-known/acme-challenge/*" {
                           root "/acme"
                           request strip 2
                   }
                   root "/htdocs"
           }

FILES
     /etc/acme-client.conf  acme-client(1) configuration file

SEE ALSO
     acme-client(1)

HISTORY
     The acme-client.conf file format first appeared in OpenBSD 6.1.

Void Linux                      August 3, 2018                      Void Linux
```
