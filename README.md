# Red Hat OpenShift GCP Lab Builder

[![Provision OpenShift Lab](https://github.com/PaloAltoNetworks/redhat-openshift-lab/actions/workflows/openshift-build.yml/badge.svg)](https://github.com/PaloAltoNetworks/redhat-openshift-lab/actions/workflows/openshift-build.yml)

Welcome to the self-service portal for provisioning ephemeral Red Hat OpenShift 4.x clusters on Google Cloud Platform (GCP). 

This repository uses GitHub Actions to automate the `openshift-install` process, allowing team members to spin up customized lab environments on demand. To prevent runaway cloud costs, **all clusters are strictly capped at a 72-hour lifespan** and will be automatically destroyed by a background Janitor process.

---

## How to Provision a Cluster

You do not need to clone this repository or run any Terraform/CLI commands locally to get a cluster.

1. Navigate to the **[Actions](../../actions)** tab at the top of this repository.
2. On the left sidebar, click on **Provision OpenShift Lab**.
3. On the right side of the screen, click the **Run workflow** dropdown button.
4. Fill out the form:
   * **Cluster Name:** Provide a unique, identifiable name (e.g., `shrey-ocp-test-1`).
   * **Worker Count:** Select the number of worker nodes you need.
   * **Acknowledgment:** You must check the box acknowledging the 72-hour automated deletion policy to proceed.
5. Click the green **Run workflow** button. 

*Note: The installation process typically takes 35 to 45 minutes to complete.*

---

## Accessing Your Cluster

Once your GitHub Actions job successfully completes, you will need to retrieve your generated credentials.

1. Click into your completed workflow run in the **Actions** tab.
2. Expand the step labeled **Output Login Credentials**.
3. Copy the **Web Console URL** and the **Password** provided in the logs.
4. Log into the OpenShift Web Console using the username `kubeadmin` and your copied password.

### CLI Access (`oc` / `kubectl`)
For security and simplicity, `kubeconfig` files are not distributed directly. To access the cluster via your local terminal:
1. Log into the OpenShift Web Console using the steps above.
2. Click your username (`kubeadmin`) in the top right corner.
3. Select **Copy login command** from the dropdown menu.
4. Paste the provided `oc login --token=...` command into your terminal.

---

## Automated Teardown (The 72-Hour Rule)

To manage GCP billing, this repository runs a continuous Janitor pipeline. 

* **Maximum Lifespan:** 72 Hours.
* **How it works:** When your cluster is built, its creation metadata is logged in a secure GCP bucket. Every 24 hours, the Janitor script checks this bucket. If your cluster is older than 72 hours, it will automatically trigger an `openshift-install destroy cluster` command.
* **Warning:** There is no backup of your lab data. Please ensure any important application manifests or configurations are committed to version control before the 72-hour window expires.

---