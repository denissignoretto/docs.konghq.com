---
title: Customize the Konnect Dev Portal
no_version: true
content_type: how-to
---


## Appearance

The Dev Portal can be customized by those with admin roles via the **Appearance
settings** in the {{site.konnect_short_name}} admin UI. To access the appearance
settings, click {% konnect_icon dev-portal %} **Dev Portal**, then **Appearance**.

From the **Appearance** menu you have the ability to modify the following options:

* Logos: default logo and favicon
* Home Page Header: welcome message, primary header, and header image
* Colors: background, text, and button colors
* Themes: default theme and dark mode
* Fonts

For details on the requirements for each customizable option, hover over the information (`i`) icon next to each item in the UI.

## Custom Dev Portal URL

Every Dev Portal instance has an auto-generated default URL. You can also manage custom URLs within {{site.konnect_short_name}}. This gives users the ability to access the Dev Portal from either the default URL, for example `https://example.us.portal.konghq.com`, or a custom URL like `portal.example.com`.

To add a custom URL to Dev Portal, you need:

* A domain and access to configure the domain's DNS `CNAME` records.
* Your organization's auto-generated default Dev Portal URL.

### Configure DNS

In your DNS configuration, create a CNAME record for the domain you want to use using the automatically generated Dev Portal URL.
The record will look like this:

| Type  | Name   | Value                                  |
|:------|--------|----------------------------------------|
| CNAME | portal | `https://example.us.portal.konghq.com` |

### Update Dev Portal URL settings {#update-portal}

To add a custom URL to Dev Portal, open {% konnect_icon dev-portal %} **Dev Portal**, click **Settings**, then follow these steps:

1. Open the **Portal URL** tab.

3. Enter the fully qualified domain name (FQDN) including the subdomain, if applicable, into the **Custom Portal URL** field.
   Don't include a path or protocol (e.g. `https://`).

4. Click **Save Custom Domain**.

5. Click **Confirm** to begin the domain verification process.

### Supported TLDs

The Custom URL feature currently supports domain names with the following top-level domains (TLDs):
`.io`, `.com`, `.tech`, `.net`, `.dev`, `.org`, `.edu`, `.gov`, `.fr`, `.se`, `.us`, `.au`, `.uk`, `.cloud`, `.site`.

If you need to use a different domain [contact support](https://support.konghq.com).

{% comment %}
### Domain name restrictions

Because of SSL certificate authority restrictions, {{site.konnect_short_name}} can't generate SSL certificates
for the following domains:

* TLDs containing a brand name: `.aws`, `.microsoft`, `.ebay`
* Hosting provider subdomains: `.amazonaws.com`, `.azurewebsites.net`
* TLDs restricted by US export laws:
  * `.af` Afghanistan
  * `.by` The Republic of Belarus
  * `.cu` Cuba
  * `.er` Eritrea
  * `.gn` Guinea
  * `.ir` Islamic Republic of Iran
  * `.kp` Democratic People's Republic of Korea
  * `.lr` Liberia
  * `.ru` The Russian Federation
  * `.ss` South Sudan
  * `.su` Soviet Union
  * `.sy` Syrian Arab Republic
  * `.zw` Zimbabwe

If you have any questions, [contact Support](https://support.konghq.com).
{% endcomment %}

### Delete a custom URL {#delete-url}

Delete a custom Dev Portal URL through your organization's {{site.konnect_short_name}} admin UI.

1. In {{site.konnect_short_name}}, open {% konnect_icon dev-portal %} **Dev Portal**, then click **Settings**.

2. Open the **Portal URL** tab.

3. Click **Delete Custom Domain**

### Troubleshoot DNS {#troubleshoot}

After the DNS verification process is complete, {{site.konnect_short_name}} will attempt to automatically generate
an SSL certificate for your custom domain. This process can take several hours. If you attempt to access the custom URL from a browser _before_ the certificate generation process is finished,
you will receive an SSL certificate error.  If this process takes more than 24 hours,
please check if the new DNS record is correctly applied. You can use the `dig` tool to troubleshoot your DNS:

* Run `dig`, replacing `CUSTOM_DOMAIN` with your custom domain
and `CUSTOM_DOMAIN_DNS` with the DNS server for the custom domain:

   ```shell
   dig +nocmd @CUSTOM_DOMAIN_DNS cname CUSTOM_DOMAIN +noall +answer
   ```

   The output will look like this:

   ```shell
   portal.example.com.	172	IN	CNAME	example.us.portal.konghq.com.
   ```

where `portal.example.com` is your custom domain and `example.us.portal.konghq.com` is the default auto-generated
URL for your Dev Portal.

{:.note}
>**Note:** If the command returns no output, or the values are incorrect, check the custom domain DNS configuration or contact your DNS provider.
>You may need to reset your Dev Portal domain settings to fix your DNS records, you can do this by [deleting the custom URL](#delete-url) and [setting the portal URL](#update-portal) again.

## Single sign-on
{:.badge .enterprise}

A {{site.konnect_short_name}} admin can configure single sign-on (SSO) for the Dev Portal via the identity settings in the {{site.konnect_short_name}} admin interface.

To configure single sign-on, open {% konnect_icon dev-portal %} **Dev Portal**, click **Settings**, then follow these steps:

1. Open the **Identity** tab.

2. Copy the callback URL and enter it in your identity provider.

2. Enter the full domain, including the subdomain and protocol, into the **Provider URL** field (also known as **Issuer**). For example, `https://accounts.google.com` for Google IdP.

3. Enter the unique identifier provided by the IdP into the **Client ID** field.

4. Enter the secret used to verify ownership of your IdP client into the **Client Secret** field.

### Configuration requirements

You must always have a form of authentication configured. Built-in and SSO can be used individually or in combination. Each state results in a different user experience, as represented in the following table:

| State | SSO registration | SSO sign in | Built-in registration| Built-in sign in | Information|
| --- | :-----------: |  ----------- | :-----------: | ----------- |
| **SSO: Enabled**<br>**Built-in: Disabled** | ✅ | ✅ | ❌ | ❌ | - Developers can register and log in with SSO.<br><br>- Developers registered with built-in authentication are forced to use SSO.<br><br>- The built-in authentication window does not display from the Dev Portal.
| **SSO: Disabled**<br>**Built-in: Enabled** | ❌ | ❌ | ✅  |✅  | - Developers can register and log in with a username and password.|
| **SSO: Enabled**<br>**Built-in: Enabled** | ✅  | ✅ | ✅  | ✅  | - Developers can register and log in with either form of authentication.|


### OIDC Details

* If a user account associated with a {{site.konnect_short_name}} developer is removed from the IdP, the {{site.konnect_short_name}} developer account is not deleted. A {{site.konnect_short_name}} admin must delete it from the {{site.konnect_short_name}} dashboard.
* If a {{site.konnect_short_name}} developer associated with an IdP user is deleted, the same IdP user can re-authenticate with the Dev Portal and a new {{site.konnect_short_name}} developer account is created. To persistently revoke access for developers authenticating through your IdP, you must remove the ability for that user to authenticate with the IdP.
* User information from the IdP is not synced with {{site.konnect_short_name}} developers after the first login.
