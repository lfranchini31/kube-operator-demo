# kube-operator-demo
Build a Kubernetes operator with Operator SDK

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
   - Modify the demo types in order to add:
      - a size attribute to DemoSpec
      - a pod array attribute to DemoStatus

      And launch 'make generate':
      ```bash
      make generate
      ```
   - Generating CRD manifests
       ```bash
       make manifests
       ```
       This makefile target will invoke controller-gen to generate the CRD manifests at config/crd/bases/demo.lfr31.com_demoes.yaml

3. Implement the Controller (Business Logic -> Core of this operator ;)!)
   - Create a demo Deployment if it doesn’t exist
   - Ensure that the Deployment foo and size are the same as specified by the Demo CR spec
   - Update the Demo CR status using the status writer with the names of the demo pods

   You can implement others features if you want ... to Infinity and Beyond!

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
      - Build and push the image
      ```bash
      # Build the image
      export REGISTRY=<dockerhub-username>
      # you need to be loggued to Docker registry -> docker login
      make docker-build IMG=$REGISTRY/demo-operator:v0.0.1
      # Push the image to the docker hub registry
      make docker-push IMG=$REGISTRY/demo-operator:v0.0.1
      ```
      - Deploy the operator
      For this demo we will run the operator in the demo namespace which can be specified for all resources in config/default/kustomization.yaml
       ```bash
       cd config/default/ && kustomize edit set namespace "demo" && cd ../..
       ```
       Install cert manager before deploy your operator
       ```bash
       kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.1.0/cert-manager.yaml
       ```
       Run the following to deploy the operator (also install the RBAC manifests from config/rbac)
       ```bash
       make deploy IMG=$REGISTRY/demo-operator:v0.0.1
       ```
       ```bash
       $ kubectl get deploy
         NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
         demo-operator-controller-manager          1/1     1            1           20s
       ```

5. Create a Demo CR
   - Update the sample Demo CR manifest at config/samples/demo_v1alpha1_demo.yaml and define the spec as the following:
   ```yaml
   apiVersion: demo.lfr31.com/v1alpha1
   kind: Demo
   metadata:
     name: demo-sample
   spec:
     foo: bar
     size: 3
   ```
   - Create the CR:
     ```bash
     kubectl apply -f config/samples/demo_v1alpha1_demo.yaml
     ```
     Ensure that the demo operator creates the deployment for the sample CR with the correct size:
     ```bash
     $ kubectl get deployment
      NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
      demo-operator-controller-manager   1/1     1            1           5m
      demo-sample                        3/3     3            3           1m
     ```
     Check the pods and CR status to confirm the status is updated with the demo pod names:
     ```bash
     $ kubectl get po
      NAME                                  READY     STATUS    RESTARTS   AGE
      demo-sample-9df5c78d7-5pmde           1/1       Running   0          1m
      demo-sample-9df5c78d7-5pmde           1/1       Running   0          1m
      demo-sample-9df5c78d7-5pmde           1/1       Running   0          1m
     ```

6. Clean-up
   - Delete the CR to uninstall Demo
   ```bash
   kubectl delete -f config/samples/demo_v1alpha1_demo.yaml
   ```
   - Uninstall the operator and its CRDs
   ```bash
   kustomize build config/default | kubectl delete -f -
   ```

(BONUS) - Create a monitoring namespace and deploy prometheus operator
   - Add bitnami helm repository
    ```bash
    kubectl create ns monitoring
    ```
    - Install the helm chart
    ```bash
    helm upgrade --install -n monitoring prometheus prometheus-community/kube-prometheus-stack
    ```


