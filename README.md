# kube-operator-demo
Build a Kuberenetes operator with Operator SDK

1. Create k3d demo-operator cluster (3 nodes)
   First, you need to install k3d distribution (allows to create containerized k3s clusters).
   ```bash
   k3d cluster create --api-port 127.0.0.1:6444 demo-operator --servers 3
   ```
2. Bootstrap a new Kubvernetes Operator
   - Project initialization
   ```bash
   operator-sdk init kube-operator-demo --domain=lfr31.com --repo github.com/lfranchini31/kube-operator-demo --license apache2 --owner "Lionel FRANCHINI"
   ```
   - Create a simple API:
    ```bash
   operator-sdk create api --group demo --version v1alpha1 --kind Demo --resource=true --controller=true
   ```
   - Generating CRD manifests
       ```bash
       make manifests
       ```
3. Implement the Controller
   - Create a demo Deployment if it doesn’t exist
   - Ensure that the Deployment foo and size are the same as specified by the Demo CR spec
   - Update the Demo CR status using the status writer with the names of the demo pods

4. Build and run the operator
   ```bash
    make install
    ```
    Once this is done, there are two ways to run the operator:
    - As Go program outside a cluster
    ```bash
    make run ENABLE_WEBHOOKS=false
    ```
    - As a Deployment inside a Kubernetes cluster
   ```bash
    
    ```


5. Clean-up
   - Delete the CR to uninstall Demo
   ```bash
   kubectl delete -f config/samples/demo_v1alpha1_demo.yaml
   ```
   - Uninstall the operator and its CRDs
   ```bash
   kustomize build config/default | kubectl delete -f -
   ```

99. Create a monitoring namespace and deploy prometheus operator
   - Add bitnami helm repository
    ```bash
    kubectl create ns monitoring
    ```
    - Install the helm chart
    ```bash
    helm upgrade --install -n monitoring prometheus prometheus-community/kube-prometheus-stack
    ```


