# Development environment

If you have [VS Code](https://code.visualstudio.com/) and the [VS Code Remote Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension installed, the `.devcontainer` folder will be used to provide a container based development environment for this repo.

We have deployed the AKS cluster with default and agent node pool on Azure cloud. We need to access the cluster setting up the kubeconfig.

Set cluster context
1. Open terminal
2. Run the following commands
Login to your azure account
az login
Set the cluster subscription
az account set --subscription <sub-id>
Download cluster credentials
az aks get-credentials --resource-group RG1-demo-AKS --name AKS-demo-nxtgenhub --overwrite-existing


## Terminal

The VS Code Terminal will run a Bash shell and make the following utilities will be available:

- curl
- azure kubernetes services
- Docker cli
- Helm 3.x
- jq
- kubectl 
- vim

The following host folders will be mounted into the VS Code dev container:

- `/var/run/docker.sock` - allow for docker builds
- `~/.kube` - allow for access to kubeconfig file

## Application development

Port `8080` has been configured to be forwarded to your host. If you run `npm start` in the `src/app` folder in the **VS Code Remote Containers** terminal, you will be able to access the website on `http://localhost:8080`. You can change the port in the `.devcontainer/devcontainer.json` file under the `appPort` key.

See the **Visual Studio Code Remote Containers** [guidance](https://code.visualstudio.com/docs/remote/containers) for more details on working with this setup.
