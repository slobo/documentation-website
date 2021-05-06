---
layout: default
title: Disable Security
parent: Configuration
grand_parent: Security
nav_order: 99
---

# Disable security

You might want to temporarily disable the security plugin to make testing or internal usage more straightforward. To disable the plugin, add the following line in `opensearch.yml`:

```yml
opensearch_security.disabled: true
```

A more permanent option is to remove the security plugin entirely. Delete the `plugins/opensearch-security` folder on all nodes, and delete the `opensearch_security` configuration entries from `opensearch.yml`.

To perform these steps on the Docker image, see [Customize the Docker image](../../../install/docker/#customize-the-docker-image).

Disabling or removing the plugin exposes the configuration index for the security plugin. If the index contains sensitive information, be sure to protect it through some other means. If you no longer need the index, delete it.
{: .warning }


## Remove OpenSearch Dashboards plugin

The security plugin is actually two plugins: one for OpenSearch and one for OpenSearch Dashboards. You can use the OpenSearch plugin independently, but the OpenSearch Dashboards plugin depends on a secured OpenSearch cluster.

If you disable the security plugin in `opensearch.yml` (or delete the plugin entirely) and still want to use OpenSearch Dashboards, you must remove the corresponding OpenSearch Dashboards plugin. For more information, see [Standalone OpenSearch Dashboards plugin install](../../../opensearch-dashboards/plugins/).


### RPM or DEB

1. Remove all `opensearch_security` lines from `opensearch_dashboards.yml`.
1. Change `opensearch.url` in `opensearch_dashboards.yml` to `http://` rather than `https://`.
1. Enter `sudo /usr/share/opensearch-dashboards/bin/opensearch-dashboards-plugin remove opensearchSecurityOpenSearch Dashboards`.
1. Enter `sudo systemctl restart opensearch-dashboards.service`.


### Docker

1. Create a new `Dockerfile`:

   ```
   FROM opensearch/opensearch-dashboards:{{site.opensearch_version}}
   RUN /usr/share/opensearch-dashboards/bin/opensearch-dashboards-plugin remove opensearchSecurityOpenSearch Dashboards
   COPY --chown=opensearch-dashboards:opensearch-dashboards opensearch_dashboards.yml /usr/share/opensearch-dashboards/config/
   ```

   In this case, `opensearch_dashboards.yml` is a "vanilla" version of the file with no OpenSearch entries. It might look like this:

   ```yml
   ---
   server.name: opensearch-dashboards
   server.host: "0"
   opensearch.hosts: http://localhost:9200
   ```


1. To build the new Docker image, run the following command:

   ```bash
   docker build --tag=opensearch-dashboards-no-security .
   ```

1. In `docker-compose.yml`, change `opensearch/opensearch-dashboards:{{site.opensearch_version}}` to `opensearch-dashboards-no-security`.
1. Change `OPENSEARCH_URL` (`docker-compose.yml`) or `opensearch.url` (your custom `opensearch_dashboards.yml`) to `http://` rather than `https://`.
1. Change `OPENSEARCH_HOSTS` or `opensearch.hosts` to `http://` rather than `https://`.
1. Enter `docker-compose up`.