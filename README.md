# Posthog

Our Posthog installation is configured by the Helm chart described in `values.yaml`. The entire installation is configured to be highly available, i.e. all deployments are replicated and have readiness and liveness probes.

> **NOTE**: The `*.secret.yaml` manifests should be deployed **before** the main chart, as there are credentials in the chart that are read from in-cluster secrets instead of being set in the chart, e.g.:
> ```
>  -- Name of an existing Kubernetes secret object containing the PostgreSQL password
>  existingSecret: posthog-postgres
>  -- Name of the key pointing to the password in your Kubernetes secret
>  existingSecretPasswordKey: password
> ```

## Deployment

To deploy PostHog, first create the necessary secrets:

```bash
kubectl apply -f postgres.secret.yaml
kubectl apply -f redis.secret.yaml
```

Then, download the charts and install them using the `values.yaml` settings:

```bash
helm repo add posthog https://posthog.github.io/charts-clickhouse/
helm repo update
helm upgrade --install -f values.yaml --timeout 30m --create-namespace --namespace posthog posthog posthog/posthog --wait --wait-for-jobs --debug
```

Further deployment instructions can be found [here](https://posthog.com/docs/self-host/deploy/aws#installing-the-chart), in PostHog's official documentation.

## Dependencies

All dependencies are configured in the `values.yaml` file - both managed and self-hosted. In the sections below, we'll describe what services fit in each category.

### Managed dependencies

PostHog's architecture involves some external databases. We are using managed AWS offerings when possible, due to its higher reliability and easier maintenance. 

In our case, these would be:

- PostgreSQL (AWS RDS)
- Kafka (AWS MSK)
- Redis (AWS ElastiCache)

### Self-hosted dependencies

These are dependencies that are running in our Kubernetes cluster and are deployed together with the PostHog chart.

- Zookeeper
- Clickhouse

## Related

- [Deploying PostHog to AWS](https://posthog.com/docs/self-host/deploy/aws)
- [PostHog Helm chart configuration](https://posthog.com/docs/self-host/deploy/configuration)
