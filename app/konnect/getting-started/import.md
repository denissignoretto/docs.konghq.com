---
title: Import Kong Gateway Entities into Konnect Cloud
no_version: true
content_type: how-to
---

If you are an existing {{site.base_gateway}} user looking to use {{site.konnect_short_name}}
as your cloud-hosted control plane, you can use [decK](/deck/) to import your
{{site.base_gateway}} entity configuration into a runtime group in your
{{site.konnect_short_name}} organization.

You can also use this guide to migrate from `konnect.konghq.com` to `cloud.konghq.com`.

Afterward, you must manually move over:
* Dev Portal files, developer accounts, and applications
* Application registrations
* Convert roles and permissions into {{site.konnect_short_name}} teams
* Certificates
* Custom plugins

You cannot import [unsupported plugins](/konnect/servicehub/plugins/#plugin-limitations).

## Prerequisites
* {{site.konnect_saas}} [account credentials](/konnect/getting-started/access-account/).
* decK v1.12 or later [installed](/deck/latest/installation/).

## Import entity configuration

Use deck to import entity configurations into a runtime group.

When you provide any {{site.konnect_short_name}} flags, decK targets the `cloud.konghq.com` environment by default.
If you want to target the `konnect.konghq.com` environment instead, use the [`--konnect-addr`](/deck/latest/guides/konnect/#target-a-konnect-api) flag.

1. Make sure that decK can connect to your {{site.konnect_short_name}} account:

    ```sh
    deck ping \
      --konnect-runtime-group-name default \
      --konnect-email {YOUR_EMAIL} \
      --konnect-password {YOUR_PASSWORD}
    ```

    If the connection is successful, the terminal displays the full name of the
    user associated with the account:

    ```sh
    Successfully Konnected as Some Name (Org Name)!
    ```

    You can also use decK with {{site.konnect_short_name}} more securely by storing
    your password in a file, then either calling it with
    `--konnect-password-file /path/{FILENAME}.txt`, or adding it to your decK configuration
    file under the `konnect-password` option along with your email:

    ```yaml
    konnect-password: {YOUR_PASSWORD}
    konnect-email: {YOUR_EMAIL}
    ```

    The default location for this file is `$HOME/.deck.yaml`. You can target a
    different configuration file with the `--config /path/{FILENAME}.yaml` flag,
    if needed.

    The following steps all use a `.deck.yaml` file to store the
    {{site.konnect_short_name}} credentials instead of flags.

1. Export configuration:

{% capture export_config %}
{% navtabs %}
{% navtab From Kong Gateway %}

Run [`deck dump`](/deck/latest/reference/deck_dump):

```sh
deck dump
```
{% endnavtab %}
{% navtab From konnect.konghq.com %}

Run [`deck dump`](/deck/latest/reference/deck_dump) and point decK at the `konnect.konghq.com` environment:

```sh
deck dump \
  --konnect-addr https://konnect.konghq.com \
  --konnect-email {YOUR_EMAIL} \
  --konnect-password {YOUR_PASSWORD}
```
{% endnavtab %}
{% endnavtabs %}
{% endcapture %}

{{ export_config | indent | replace: " </code>", "</code>" }}

    This command outputs {{site.base_gateway}}'s object configuration into
    `kong.yaml` by default. You can also set `--output-file /path/{FILENAME}.yaml`
    to set a custom filename or location.

1. Open the file. If you have any of the following in your configuration, remove it:

    * Any `_workspace` entries: There are no workspaces in {{site.konnect_short_name}}. For a similar
    concept, see [runtime groups](/konnect/runtime-manager/runtime-groups).

    * Configuration for the Portal App Registration plugin: App registration is
    [supported in {{site.konnect_short_name}}](/konnect/dev-portal/applications/application-overview),
    but not through a plugin, and decK does not manage it.

    * Any other unsupported plugins:
        * OAuth2 Authentication
        * Apache OpenWhisk
        * Vault Auth
        * DeGraphQL
        * GraphQL Rate Limiting Advanced
        * Key Authentication Encrypted

1. Preview the import with the [`deck diff`](/deck/latest/reference/deck_diff)
command, pointing to the runtime group that you want to target:

    ```sh
    deck diff --konnect-runtime-group-name default
    ```

    If you're not using the default `kong.yaml` file, specify the filename and
    path with `--state /path/{FILENAME}.yaml`.

1. If you're satisfied with the preview, run [`deck sync`](/deck/latest/reference/deck_sync):

    ```sh
    deck sync --konnect-runtime-group-name default
    ```

    If you don't specify the `--konnect-runtime-group-name` flag, decK targets the
    `default` runtime group. If you have more than one runtime group in your
    organization, we recommend always setting this flag to avoid accidentally
    pushing configuration to the wrong group.

1. Log in to your [{{site.konnect_saas}}](http://cloud.konghq.com/login) account.

1. From the left navigation menu, open **Runtime Manager**, then open the runtime group
you just updated.

1. Look through the configuration details of any imported entities to make sure
they were migrated successfully.

## Migrate data planes

You can keep any data plane nodes that are:
* Running {{site.base_gateway}} (Enterprise, include _free_ mode)
* Are at least version 2.5 or higher

Turn any self-managed nodes into cloud data plane nodes by registering them
through the Runtime Manager and adjusting their configurations, or power down
the old instances and create new data plane nodes through {{site.konnect_saas}}.

1. Follow the [runtime setup guide](/konnect/runtime-manager/#runtime-instances) for
your preferred deployment type.

2. Once you have created or converted the data plane nodes, `kong stop` your
old Gateway runtimes, then shut them down.

3. If any of the old nodes have connected PostgreSQL or Cassandra instances,
you can shut them down now.

## Post-migration tasks

See the following docs to set up any additional things you may need:

* **Dev Portal files:** You can migrate API specs and markdown service descriptions
into Service Hub using the {{site.konnect_saas}} GUI. Each {{site.konnect_short_name}} service accepts
one markdown description file, and each service version accepts one API spec.
See [Dev Portal Service Documentation](/konnect/servicehub/service-documentation).

* **Dev Portal applications and developers:** If you have developers or
applications registered through the Portal, those developers need to create new
accounts in {{site.konnect_saas}} and register their applications in the new
location.
    * [Create Dev Portal accounts](/konnect/dev-portal/dev-reg)
    * [Enable application registration](/konnect/dev-portal/applications/enable-app-reg):
    App registration in {{site.konnect_saas}} works through a different
    mechanism than in self-managed {{site.base_gateway}}. Enable app
    registration on each service that requires it.
    * [Publish services to the Dev Portal](/konnect/servicehub/service-documentation/#publishing):
    The Dev Portal is automatically enabled on a {{site.konnect_saas}} org
    (Plus or Enterprise tier). Publish your services to the Dev Portal.
* [**Prepare custom plugins for migration**](/konnect/servicehub/plugins/#custom-plugins):
Custom plugins are supported in {{site.konnect_saas}}, but with limitations. As
long as your plugins fit the criteria, or if you can adjust them to do so,
contact Kong Support to get the plugin manually added to your account.
* [**Review and set up teams and roles**](/konnect/org-management/teams-and-roles):
{{site.konnect_saas}} groups and roles don't map directly to
{{site.base_gateway}} teams and roles. Set up teams to mirror your
{{site.base_gateway}} groups, then invite users to your {{iste.konnect_saas}}
org and assign them to a team on invite.
