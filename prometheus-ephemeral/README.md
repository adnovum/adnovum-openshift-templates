A Prometheus server to be deployed from within your project, and that
doesn't require the "cluster-reader" role. Super easy to deploy.

The deployed Prometheus instance does service discovery of your project
resources, and automatically polls your annotated metrics endpoints.
Also, a blackbox_exporter is used for probing of services, and
kube-state-metrics is configured to get "state" metrics about your
objects (like number of pods, etc.).

Prometheus uses EmptyDir for storage, and retains metrics only for 2
hours. It is meant to be used only as temporary storage for the metrics,
which can be federated from a bigger Prometheus instance outside of the
OpenShift project for long-term storage, alerting, visualization, etc.

Authentication / authorization is not implemented. You are supposed to
protect the exposed prometheus URL.

A "view" RoleBinding in the project is created and is required for
service discovery.
