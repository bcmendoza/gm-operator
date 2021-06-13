# Grey Matter Operator

Grey Matter Operator is a Kubernetes operator that watches for install.greymatter.io/v1.Mesh CR (Custom Resource) objects in a Kubernetes cluster and installs the core Grey Matter services into the same namespace where the Mesh CR exists. Grey Matter Operator also spawns a process that will call the Control API to configure routing for each installed core service.

## Important Files

- [api/v1/mesh_types.go](api/v1/mesh_types.go): Used to generate the Mesh CRD. Every time this is updated, run `make generate` and `make manifests`.
- [config/crd/bases/install.greymatter.io_meshes.yaml](config/crd/bases/install.greymatter.io_meshes.yaml): The Mesh CRD generated by `make manifests`.
- [controllers/mesh_controller.go](controllers/mesh_controller.go): Logic for what to do when a Mesh CR is created/updated (e.g. deploy and configure the mesh).
- [controllers/reconciler.go](controllers/reconciler.go): A custom interface with methods for reconciling Kubernetes resources based on configuration passed in from a Mesh CR. See the `reconcilers` directory for how this interface is implemented for each Kubernetes resource.

## Getting Started

This assumes you have at least Go 1.15, K3d, and kubectl installed.

1. Download Go dependencies: `go mod vendor`
2. Build the Docker image: `make docker-build`
3. Push the Docker image to Nexus: `make docker-push`
4. Make a K3d cluster: `k3d cluster create gm-operator -a 1 -p 30000:10808@loadbalancer`
5. Set your `KUBECONFIG` to it: `export KUBECONFIG=$(k3d kubeconfig write gm-operator)`
6. Store your Grey Matter LDAP credentials in the environment variables `NEXUS_USER` and `NEXUS_PASSWORD` and then run the create script to create a secret for pulling Docker images: `./create-docker-secret.sh`

*NOTE: The Docker secret is created in the `default` namespace for now, although later on we'd want to create it in the `gm-operator-system` namespace so that the Operator can re-create the secret in each namespace where a Mesh CR is deployed.*

Then to deploy:

1. Deploy to the K3d cluster: `make deploy`.
2. Create a Mesh CR: `kubectl apply -f config/samples/install_v1_mesh.yaml`
3. Check the Mesh CR: `kubectl get mesh sample-mesh -o yaml`

## Cleanup

1. `kubectl delete -f config/samples/install_v1_mesh.yaml`
2. `make undeploy`
3. `k3d cluster delete gm-operator`

## Resources

- [Operator Framework: Go Operator Tutorial](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/)
- [Operator SDK Installation](https://sdk.operatorframework.io/docs/building-operators/golang/installation/)
- [Operator Manager Overview](https://book.kubebuilder.io/cronjob-tutorial/empty-main.html)
- [Istio's operator spec](https://github.com/istio/api/blob/master/operator/v1alpha1/operator.pb.go#L97)
