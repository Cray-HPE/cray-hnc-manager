cray-hnc-manager
-------

See <https://github.com/kubernetes-sigs/hierarchical-namespaces/releases/> for info on latest release.

Steps to update this chart:

1. First update the container image.  Example:
   * <https://github.com/Cray-HPE/container-images/commit/6dc6f340d1d8a78018801cf59b03f0c1fd4255aa>

   ```
   ./.github/scripts/create_buildfiles.sh -r gcr.io -i k8s-staging-multitenancy/hnc-manager -t v1.0.0
   ```

   * update the container version in values.yaml

1. Bring in the new hnc template:

   * In the templates dir:

     ```
     HNC_VERSION=v1.0.0-rc2
     curl -o hnc-manager-${HNC_VERSION}.yaml https://github.com/kubernetes-sigs/hierarchical-namespaces/releases/download/${HNC_VERSION}/default.yaml
     ```

   * comment out the namespace creation at the top (cray-drydock does that)
   * add back this section for the manager args so it honors customizations:

     ```
     timeoutSeconds: {{ .Values.webhookTimeoutSeconds }}
     ```

     ```
     replicas: {{ .Values.numReplicas }}
     ```

     ```
     {{- if .Values.validTenantNamePrefix }}
     - --included-namespace-regex=(^multi-tenancy$|^tenants$|^tapms-operator$|^slurm-operator$|^{{ .Values.validTenantNamePrefix }}.*)
     {{- else }}
     - --included-namespace-regex=(^multi-tenancy$|^tenants$|^tapms-operator$|^slurm-operator$)
     {{- end }}
     ```

   * remove the old version (hnc-manager-v0.9.0.yaml)
