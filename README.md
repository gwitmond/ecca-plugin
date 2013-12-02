ecca-plugin
===========

Firefox Plug In for the Eccentric Authentication protocol

Eccentric Authentication uses locally signed client certificates to
eliminate passwords, prevent passive and active attacks. Makes
encryption easy to use.

# Functional Design

## Login and Logout handing

User signup handles the aspects of creating a public/private key pair,
requesting a certificate and actually logging in and out of the site.

### Log in with an existing account

- user browses to a web site;
- plug in validates server certificate;
- plug in presents the list of accounts it has for this site;
- user selects one of the accounts and plug in presents the matching client certificate to the server.

### Log out

- user presses the logout button at the plug in (not at the site)
- plug in stops using client certificates for connections to the site;
- plug in deletes all cookies and other session related state.

### Create new account:

 - user browses to a web site;
 - plug in validates server certificate;
 - site requires a client certificate for certain access; site specifies the url of the local CA it trusts;
 - plug in detects the requirement and shows the login/singup menu
 - user specifies an account name; types it into plug in (or presses 'random account name'-button);
 - plug in generates a public/private key pair; 
 - plug in requests a certificate at the local CA; and retrieves a certificate
 - plug in stores the certifcate, together with information for which site it is valid;
 - plug in will present this certificate to the web server at next connection. (auto login).

### Delete account

- plug in shows list of sites with all accounts at that site;
- user selects some account for deletion;
- plug in removes the private key of the selected accounts; web site will not be wiser;

Deleting a private key is the sole action to take when deleting an
account. The site may never learn of it. The site will never see that
account log in again. It may delete data attached to that account
after a certain time. Or it could show a timer value: last log in at
<date>. If it gets too old, other people may assume an abandoned
account.

## Private Message handling

Users can send private messages to other users. Messages are encrypted
using the recipients public key. Users learn about other peoples' keys
via websites. By reading a signed blog entry, and verifying the
signature, you learn the public key of the author.

As the combination of {accountname, sitename} is always unique
(property of eccentric authentication), people can communicate their
address to others. They must verify uniqueness. If it is unique, you
can lookup the corresponding certificate. Inside the certificate lies
the public key of your correspondent.

### Sending a private message on a blog/forum/webmail-site

Assume user has an account.

- user browses to the compose-message page
- user specifies recipient;
- plug in fetches certificate
- plug in validates uniqueness;
- user composes a message;
- optional: plug in signs the message with the users' private key (for the current active account);
- plug in encrypts it with the recipients public key
- plug in hands it to the site;

### Reading a private message on a blog/forum/webmail-site

- user browses to the site;
- user logs in with their client certificate;
- site shows page with encrypted messages;
- plug in decodes the messages;

## Posting signed public messages

Users can post messages to blog sites, these messages are signed with
the user's private key. This serves two purposes:
- it validates to the user that the site is not messing up with the message; nor performing a MitM;
- it ties the public/private key pair to the message.

The second part, effectively distributes the public key to the
readers. Readers can use it to write a private message to the author.

Assume the user has an account:
- user logs in at the site
- user browses to the post-a-public-message page;
- user composes the message;
- plug in signs the message with the private key of the current active account;
- plug in hands the signed message to the site for posting.

# Validations

The Plug In performs validations at certain points in the protocol.

### Validating server certificate

When the plug in connects to a site, it needs the Root Certificate of the sites' localCA. 
The Root Certificate is then used to validate the server certificate. 

- user wants to browse to a web site;
- plug in looks up DANE records in DNSSEC for site;
- plug in validates DNSSEC delegation against the well known DNSSEC root key (pinned);
- plug in uses the Root Certificate in the TLS configuration as the single trusted CA; 
- plug in lets normal browsing go on.



