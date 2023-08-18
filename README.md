# guestbook-harness-demo

This tutorial will guide you through deploying a sample application on Kubernetes using a continuous deployment pipeline. We'll use the Harness CD pipeline to fetch an image from a registry, execute a canary deployment, and then perform a rolling deployment to launch the `guestbook` application on a Kubernetes cluster.

This tutorial assumes that the reader has foundational knowledge of Kubernetes and GitOps, or a more advanced understanding. Here is a [related resource](https://www.harness.io/learn/use-cases/kubernetes).

## Before you begin

In order to follow this tutorial, you need some prerequisites.

1. [Fork this repository](https://github.com/dewandemo/guestbook-harness-demo/fork).
2. [Sign up for free on Harness platform](https://app.harness.io/auth/#/signup/?module=cd&utm_source=github&utm_medium=github-tutorial&utm_campaign=dewan-devrel).
3. Create a GitHub personal access token with the repo scope. [Fine-grained personal access token (in beta at the time of writing this tutorial)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) allows you many advantages over the [classic personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic). Be sure to save the token after creating it since you won't be able to see it again.
4. Have a self-hosted or managed Kubernetes cluster. We recommend [k3d](https://k3d.io/) for installing Harness Delegates and deploying a sample application in a local development environment. Read more about [Harness delegate](https://developer.harness.io/docs/platform/delegates/delegate-concepts/delegate-overview/) and [delegate system requirements](https://developer.harness.io/docs/platform/Delegates/delegate-concepts/delegate-requirements).
5. [Sign up for a docker hub account](https://hub.docker.com/) to be able to push and manage container images. You can use any other image registry. For this tutorial, we'll be using docker hub.
6. Install the [Helm CLI](https://helm.sh/docs/intro/install/) in order to install the Harness Helm delegate.

> [!WARNING]  
> For the pipeline to run successfully, please follow the remaining steps as they are, including the naming conventions.

## Configuring Harness CD

1. Log in to [Harness](https://app.harness.io/).
2. You might already be under **Default Project**. If not, switch to **Default Project** from **Projects** section.

### Delegate installation

<details>
<summary>What is the Harness delegate?</summary>
<br>
The Harness delegate is a service that runs in your local network or VPC to establish connections between the Harness Manager and various providers such as artifacts registries, cloud platforms, etc. The delegate is installed in the target infrastructure, for example, a Kubernetes cluster, and performs operations including deployment and integration. Learn more about the delegate in the <a href=https://developer.harness.io/docs/platform/delegates/delegate-concepts/delegate-overview>Delegate overview</a>.
</details>

Under **Project Setup**, select **Delegates**. If there's no existing delegate, you can select **Install a delegate**. For subsequent ones, choose **New Delegate**.

For this tutorial, let's explore how to install a delegate using Helm.

> [!NOTE]  
> Ensure you're connected to a Kubernetes cluster before running the commands below.

1. Add the Harness Helm chart repo to your local helm registry using the following commands.

```shell
helm repo add harness-delegate https://app.harness.io/storage/harness-download/delegate-helm-chart/
```

2. Update the repo:

```shell
helm repo update harness-delegate
```

3. In the following example command, ACCOUNT_ID and MANAGER_ENDPOINT are auto-populated values that you can obtain from the delegate installation wizard. Be sure to use the latest `delegateDockerImage` value which can also be found from the delegate installation wizard.

```shell
helm upgrade -i helm-delegate --namespace harness-delegate-ng --create-namespace \
harness-delegate/harness-delegate-ng \
 --set delegateName=helm-delegate \
 --set accountId=ACCOUNT_ID \
 --set managerEndpoint=MANAGER_ENDPOINT \
 --set delegateDockerImage=harness/delegate:23.03.78904 \
 --set replicas=1 --set upgrader.enabled=false \
 --set delegateToken=DELEGATE_TOKEN
```

4. Select **Verify** to verify that the delegate is installed successfully and can connect to the Harness Manager.

### Secrets

<details>
<summary>What are Harness secrets?</summary>
<br>
Harness offers built-in secret management for encrypted storage of sensitive information. Secrets are decrypted when needed, and only the private network-connected Harness delegate has access to the key management system. You can also integrate your own secret manager. To learn more about secrets in Harness, go to <a href=https://developer.harness.io/docs/platform/Secrets/Secrets-Management/harness-secret-manager-overview/>Harness Secret Manager Overview</a>.
</details>

Under **Project Setup**, select **Secrets**.

1. Create Harness secret for GitHub.

- Select **New Secret**, and then select **Text**.
- Enter the secret name `harness_gitpat`.
- For the secret value, paste the GitHub personal access token you saved earlier.
- Select **Save**.

2. Create Harness secret for Docker Hub.

- Select **New Secret**, and then select **Text**.
- Enter the secret name `docker_secret`.
- For the secret value, either use your docker hub password or [access token](https://docs.docker.com/docker-hub/access-tokens/) (recommended).
- Select **Save**.

### Connectors

<details>
<summary>What are connectors?</summary>
<br>
Connectors in Harness enable integration with 3rd party tools, providing authentication and operations during pipeline runtime. For instance, a GitHub connector facilitates authentication and fetching files from a GitHub repository within pipeline stages. Explore connector how-tos <a href=https://developer.harness.io/docs/category/connectors/>here</a>.
</details>

In your Harness project in the Harness Manager, under **Project Setup**, select **Connectors**.

1. Create the **GitHub connector**.

   - Copy the contents of [github-connector.yml](harnesscd-pipeline/github-connector.yml).
   - Click **New Connector** under **Connectors**.
   - Select **Create via YAML Builder** and paste the copied YAML.
   - Assuming you have already forked this repository, replace `GITHUB_USERNAME` with your GitHub account username in the YAML.
   - In `projectIdentifier`, verify that the project identifier is correct. You can see the Id in the browser URL (after `account`). If it is incorrect, the Harness YAML editor will suggest the correct Id.
   - Select **Save Changes** and verify that the new connector named **harness_gitconnector** is successfully created.
   - Finally, select **Connection Test** under **Connectivity Status** to ensure the connection is successful.

2. Create the **Docker connector**.

   - Copy the contents of [docker-connector.yml](harnesscd-pipeline/docker-connector.yml).
   - Click **New Connector** under **Connectors**.
   - Select **Create via YAML Builder** and paste the copied YAML.
   - Replace `username` with your Docker Hub username in the YAML.
   - Replace `YOUR_ACCOUNT_ID` with your Harness account ID. You can obtain your Harness account ID from the URL in your browser when you are logged into Harness `https://app.harness.io/ng/account/<YOUR_ACCOUNT_ID>`.
   - In `projectIdentifier`, verify that the project identifier is correct. You can see the Id in the browser URL (after `account`). If it is incorrect, the Harness YAML editor will suggest the correct Id.
   - Select **Save Changes** and verify that the new connector named **harness_docker_connector** is successfully created.
   - Finally, select **Connection Test** under **Connectivity Status** to ensure the connection is successful.

3. Create the **Kubernetes connector**.
   - Copy the contents of [kubernetes-connector.yml](harnesscd-pipeline/kubernetes-connector.yml).
   - Click **New Connector** under **Connectors**.
   - Select **Create via YAML Builder** and and paste the copied YAML.
   - Replace **DELEGATE_NAME** with the installed Delegate name. To obtain the Delegate name, navigate to **Project Setup**, and then **Delegates**.
   - Select **Save Changes** and verify that the new connector named **harness_k8sconnector** is successfully created.
   - Finally, select **Connection Test** under **Connectivity Status** to verify the connection is successful.
