# guestbook-harness-demo

This tutorial will guide you through deploying a sample application on Kubernetes using a continuous deployment pipeline. We'll use the Harness CD pipeline to fetch an image from a registry, execute a canary deployment, and then perform a rolling deployment to launch the `guestbook` application on a Kubernetes cluster.

This tutorial assumes that the reader has foundational knowledge of Kubernetes and GitOps, or a more advanced understanding. Here is a [related resource](https://www.harness.io/learn/use-cases/kubernetes).

## Before you begin

In order to follow this tutorial, you need some prerequisites.

1. [Sign up for free on Harness platform](https://app.harness.io/auth/#/signup/?module=cd&utm_source=github&utm_medium=github-tutorial&utm_campaign=dewan-devrel).
2. Create a GitHub personal access token with the repo scope. [Fine-grained personal access token (in beta at the time of writing this tutorial)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) allows you many advantages over the [classic personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic). Be sure to save the token after creating it since you won't be able to see it again.
3. Have a self-hosted or managed Kubernetes cluster. We recommend [k3d](https://k3d.io/) for installing Harness Delegates and deploying a sample application in a local development environment. Read more about [Harness delegate](https://developer.harness.io/docs/platform/delegates/delegate-concepts/delegate-overview/) and [delegate system requirements](https://developer.harness.io/docs/platform/Delegates/delegate-concepts/delegate-requirements).
4. [Sign up for a docker hub account](https://hub.docker.com/) to be able to push and manage container images. You can use any other image registry. For this tutorial, we'll be using docker hub.
5. Install the [Helm CLI](https://helm.sh/docs/intro/install/) in order to install the Harness Helm delegate.

> [!WARNING]  
> For the pipeline to run successfully, please follow the remaining steps as they are, including the naming conventions.

## Configuring Harness CD

1. Log in to [Harness](https://app.harness.io/).
2. You might already be under **Default Project**. If not, switch to **Default Project** from **Projects** section.

<details>
<summary>What is the Harness delegate?</summary>
<br>
The Harness delegate is a service that runs in your local network or VPC to establish connections between the Harness Manager and various providers such as artifacts registries, cloud platforms, etc. The delegate is installed in the target infrastructure, for example, a Kubernetes cluster, and performs operations including deployment and integration. Learn more about the delegate in the <a href=https://developer.harness.io/docs/platform/delegates/delegate-concepts/delegate-overview>Delegate overview</a>.
</details>
