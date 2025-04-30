# Upgrading to {{{ docsVersionInfo.k0rdentName}}} 0.3.0

Several changes in 0.3.0 can require manual intervention while upgrading and
described in their respective sections below.

## `defaultRegistryURL` parameter rename

In {{{ docsVersionInfo.k0rdentName}}} 0.3.0 KCM controller parameter
`defaultRegistryURL` was renamed to `templatesRepoURL`.

If you used a custom value for the `defaultRegistryURL` parameter upon upgrade
add `templatesRepoURL` with the same value to the
`spec.core.kcm.config.controller` of the `Management` object. After upgrade to
v0.3.0 the `defaultRegistryURL` parameter can be removed completely.

## Possible projectsveltos upgrade issue

Possible issue can arise when updating `projectsveltos` component. In case of
issue upgrade is stuck with an error regarding immutable field when patching Job
`register-mgmt-cluster-job`.

Example of the error:

```
Helm upgrade failed for release projectsveltos/projectsveltos with chart projectsveltos@0.52.2: cannot patch "register-mgmt-cluster-job" with kind Job: Job.batch "register-mgmt-cluster-job" is invalid: spec.template: Invalid value
```

To mitigate this error the following steps must be done:

1. Any jobs in the `projectsveltos` namespace must be deleted
   ```bash
   kubectl -n projectsveltos delete jobs --all
   ```
1. The `projectsveltos` helm release must be remediated with the following command.
	```bash
	TOKEN="$(date +%s)" kubectl -n
	kcm-system annotate hr projectsveltos "reconcile.fluxcd.io/requestedAt=$TOKEN"
	"reconcile.fluxcd.io/forceAt=$TOKEN"
	```
