# Install microk8s

    snap install microk8s --classic --channel=1.17/stable
    sudo usermod -a -G microk8s $USER
    echo 'export PATH=$PATH:/snap/bin' >> ~/.zshrc

To let MicroK8s use a proxy we need to enter the proxy details in \${SNAP_DATA}/args/containerd-env (normally /var/snap/microk8s/current/args/containerd-env). The containerd-env file holds the environment variables containerd runs with. Setting the HTTPS_PROXY to your proxy endpoint enables containerd to fetch conatiner images from the web

check status

    microk8s.status --wait-ready

MicroK8s bundles its own version of kubectl for accessing Kubernetes. Use it to run commands to monitor and control your Kubernetes.

microk8s.kubectl get nodes
microk8s.kubectl get services

MicroK8s uses a namespaced kubectl command to prevent conflicts with any existing installs of kubectl. If you don't have an existing install, it is easier to add an alias

    alias kubectl='microk8s.kubectl'

MicroK8s uses the minimum of components for a pure, lightweight Kubernetes. However, plenty of extra features are available with a few keystrokes using "add-ons" – pre-packaged components that will provide extra capabilities for your Kubernetes, from simple DNS management to machine learning with Kubeflow!

To start it is recommended to add DNS management to facilitate communication between services. For applications which need storage, the 'storage' add-on provides directory space on the host. These are easy to set up

    microk8s.enable dns storage

Full list of addons

    https://microk8s.io/docs/addons

Starting and stopping

    microk8s.stop

    microk8s.start
