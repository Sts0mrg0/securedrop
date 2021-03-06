# SecureDrop Deployment Best Practices

SecureDrop is only as secure as the environment that surrounds it. To keep sources safe, the news organization's website must employ a set of basic security best practices or else you risk losing any source protection provided by SecureDrop.

While SecureDrop itself is located on a Tor hidden service, news organizations also need to create a SecureDrop landing page that will explain how SecureDrop works, give sources instructions on how to access the Tor hidden service, and disclose the risks.

It is important to keep in mind that implementing SecureDrop will bring more attention to your organization by security researchers, hackers, and other like-minded people. If the landing page minimum requirements are not implemented, these people will be sure to loudly point this out on Twitter and other blogs as a #SecurityFail. This will discourage potential sources from using your instance of SecureDrop. However, it can easily be avoided by following the below best practices.

Freedom of the Press Foundation eventually plans to [list all of those SecureDrop onion URLs](https://freedom.press/securedrop/directory) as "verified" on its website that meet the minimum requirements for deployment best practices. If your organization cannot follow the minimum guidelines we cannot recommend to users that your SecureDrop instance is safe to use.

In addition to implementing the below best practices, it is strongly recommended that you have a reputable security firm perform a security review of your organization's public website prior to launching an instance of SecureDrop. Upon request, we can help put you in touch with a few security firms if you need more assistance.

### Landing Page

**URL and location**

Ideally you would not use a separate subdomain, but would use a path at your top-level domain, e.g. organization.com/securedrop. This is because TLS does not encrypt the hostname, so a SecureDrop user whose connection is being monitored would be trivially discovered.

**HTTPS only (no mixed content)**

Most news organizations, [in fact almost all](https://freedom.press/blog/2014/09/after-nsa-revelations-why-arent-more-news-organizations-using-https), do not use HTTPS encryption by default. This is the number one minimum requirement for the SecureDrop landing page on your website. Without HTTPS, a source can easily be exposed as a visitor to your site.

This may be difficult if your website serves advertisements or utilizes a legacy content delivery network. You should make sure the SecureDrop landing page does not serve ads of any kind, even if the rest of your site does.

If you do not serve ads on any of your site, you should also consider switching your whole site over to HTTPS by default immediately. If you do serve ads, consider pressuring your ad networks to enable you to switch to HTTPS for your entire website in the future.

If your website needs to operate in both HTTPS and HTTP mode, use protocol-relative URLs for resources such as images, CSS and JavaScript in common templates to ensure your page does not end up in a mixed HTTPS/HTTP state.

**Perfect Forward Secrecy**

Perfect Forward Secrecy (PFS) is a property of encryption protocols that ensures each SSL session has a unique key, meaning that if the key is compromised in the future it can't be used to decrypt previously recorded SSL sessions. You may need to talk to your CA (certificate authority) and CDN (content delivery network) for this, although our recommended configuration below provides forward secrecy.

**Don't load any resources (scripts, web fonts, etc.) from 3rd parties (e.g. Google Web Fonts)**

This will potentially leak information about sources to third parties, which can more easily be accessed by law enforcement agencies. Simply copy them to your server and serve them yourself to avoid this problem.

**Don't use 3rd party analytics, tracking, or advertising**

Most news websites, even those that are non-profits, use 3rd-party analytics tools or tracking bugs on their websites. It is vital that these are disabled for the SecureDrop landing page.

Both the New Yorker and Forbes were heavily criticized when launching their version of SecureDrop because their landing page contained trackers. They were claiming they were going to great lengths to protect source's anonymity, but by having trackers on their landing page, also opened up multiple avenues for third parties to collect information on those sources. This information can potentially be accessed by law enforcement or intelligence agencies and could unduly expose a source.

**Apply Applicable Security Headers**

Security headers give instructions to the web browser on how to handle requests from the web application. These headers set strict rules for the browser and help mitigate against potential attacks. Given the browser is a main avenue for attack, it is important these headers are as strict as possible.

You can use the site [securityheaders.com](https://securityheaders.com) to easily test your website's security headers.

If you use Apache, you can use these:

    Header set Cache-Control "max-age=0, no-cache, no-store, must-revalidate"
    Header edit Set-Cookie ^(.*)$ $;HttpOnly
    Header set Pragma "no-cache"
    Header set Expires "-1"
    Header always append X-Frame-Options: DENY
    Header set X-XSS-Protection: "1; mode=block"
    Header set X-Content-Type-Options: nosniff
    Header set X-Content-Security-Policy: "default-src 'self'"
    Header set X-Download-Options: noopen
    Header set X-Permitted-Cross-Domain-Policies: master-only
    Header set Content-Security-Policy: "default-src 'self'"

**Additional Apache Configuration**

To enforce HTTPS/SSL always, you need to set up redirection within the HTTP (port 80) virtual host:

    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule (.*) [](https://%)https://%{HTTP_HOST}%{REQUEST_URI}
    
In your SSL (port 443) virtual host, set up HSTS and use these settings to give preference to the most secure cipher suites:

    Header set Strict-Transport-Security "max-age=16070400; includeSubDomains"
    SSLProtocol all -SSLv2 -SSLv3
    SSLHonorCipherOrder on
    SSLCompression off
    SSLCipherSuite EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:EECDH+RC4:RSA+RC4:!MD5

You'll need to run `a2enmod headers ssl rewrite` for all these to work. You should also set `ServerSignature Off` and `ServerTokens Prod`, typically in /etc/apache2/conf.d/security.

If you use Nginx, [you can follow this link](https://gist.github.com/mtigas/8601685) and use the configuration file provided by ProPublica.

**Change detection monitoring for the web application configuration and landing page content**

OSSEC is FOSS host based intrusion detection suite that includes a file integrity monitor. More information can be found [here.](https://ossec.net)

**Don't log access to the landing page in the webserver**

Here's an Apache example that would exclude the landing page from logging:

    SetEnvIf Request_URI "^/securedrop$" dontlog
    CustomLog logs/access_log common env=!dontlog

### Minimum requirements for the SecureDrop environment

*   The Document and Monitor servers should be dedicated physical machines, not virtual machines.
*   A trusted location to host the servers. The servers should be hosted in a location that is owned or occupied by the organization to ensure that their legal can not be bypassed with gag orders.
*   The SecureDrop servers should be on a separate internet connection or completely segmented from corporate network.
*   All traffic from the corporate network should be blocked at the SecureDrop's point of demarcation.
*   Video monitoring should be recorded of the server area and the organizations safe.
*   Journalist should ensure that while using the air-gapped viewing station they are in an area without video cameras.
*   An established monitoring plan and incident response plan. Who will receive the OSSEC alerts and what their response plan will be? These should cover technical outages and a compromised environment plan.

### Suggested

*   For publicly advertised SecureDrop instances display the Source Interface's hidden service onion address on all of the organization public pages.
*   Mirror the Tor Browser and Tails so sources do not have to visit [torproject.org](torproject.org) to download it.

## Whole Site Changes

Ideally, some or all of the following changes are made to improve the overall security of the path to the landing page and obfuscate traffic analysis.

1.  Make the entire site available under 'ssl.washingtonpost.com' (ideally without the '.ssl' prefix).
 * That way, the domain won't be as suspicious as it is right now. I suspect that this is more or less the only content hosted on the domain.
2.  Include an iframe for all (or a random subset of) visitors, loading this particular URL (hidden).
 * By artificially generating traffic to the endpoint it will be harder to distinguish these from other, 'real' requests.
 * Use a random delay for adding the iframe (otherwise the 'pairing' with the initial HTTP request may distinguish this traffic).
3.  Print the link, URL and info block on the dead trees (the paper), as others have suggested.
4.  Add [HSTS headers](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security).
