# Azure DNS module for Caddy

This package contains a DNS provider module for [Caddy](https://github.com/caddyserver/caddy). It can be used to manage DNS records in Azure DNS Hosted zones.

## Caddy module name

```text
dns.providers.azure
```

## Authenticating

This module supports authentication using **a service principal with a secret**, **a workload identity**, and **a managed identity**.

See [the associated README in the libdns package](https://github.com/libdns/azure#authenticating) for important information about credentials.

## Building

To compile this Caddy module, follow the steps describe at the [Caddy Build from Source](https://github.com/caddyserver/caddy#build-from-source) instructions and import the `github.com/caddy-dns/azure` plugin

## Config examples

> [!TIP]
> These examples are for authenticating using **a service principal with a secret**.
> To attempt to authenticate using **a workload identity** or **a managed identity**, remove all of `tenant_id`, `client_id`, and `client_secret`.
> Refer to [the associated README in the libdns package](https://github.com/libdns/azure#authenticating) for more information.

To use this module for the ACME DNS challenge, [configure the ACME issuer in your Caddy JSON](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuers/acme/) like so:

```json
{
  "module": "acme",
  "challenges": {
    "dns": {
      "provider": {
        "name": "azure",
        "subscription_id": "{env.AZURE_SUBSCRIPTION_ID}",
        "resource_group_name": "{env.AZURE_RESOURCE_GROUP_NAME}",
        "tenant_id": "{env.AZURE_TENANT_ID}",
        "client_id": "{env.AZURE_CLIENT_ID}",
        "client_secret": "{env.AZURE_CLIENT_SECRET}"
      }
    }
  }
}
```

or with the Caddyfile:

```text
tls {
  dns azure {
    subscription_id {$AZURE_SUBSCRIPTION_ID}
    resource_group_name {$AZURE_RESOURCE_GROUP_NAME}
    tenant_id {$AZURE_TENANT_ID}
    client_id {$AZURE_CLIENT_ID}
    client_secret {$AZURE_CLIENT_SECRET}
  }
}
```

You can replace `{$*}` or `{env.*}` with the actual values if you prefer to put it directly in your config instead of an environment variable.

### Workload identity or managed identity

Leave out `tenant_id`, `client_id`, and `client_secret`:

```text
tls {
  dns azure {
    subscription_id {$AZURE_SUBSCRIPTION_ID}
    resource_group_name {$AZURE_RESOURCE_GROUP_NAME}
  }
}
```

A workload identity is used if the environment variables `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, and `AZURE_FEDERATED_TOKEN_FILE` are set, and then a managed identity is attempted.
Assign the **DNS Zone Contributor** role to the identity for the DNS zone.

> [!WARNING]
> With [Microsoft Entra Workload ID on AKS](https://learn.microsoft.com/en-us/azure/aks/workload-identity-overview), `AZURE_TENANT_ID` and `AZURE_CLIENT_ID` are set on your pods automatically.
> Do **not** write `tenant_id {$AZURE_TENANT_ID}` or `client_id {$AZURE_CLIENT_ID}` in your config in this case.
> If any of `tenant_id`, `client_id`, and `client_secret` is not empty, authentication using a service principal with a secret is attempted, and it fails because `client_secret` is empty.
