# Red Hat OpenShift GCP Lab Builder

[![Provision OpenShift Lab](https://github.com/PaloAltoNetworks/redhat-openshift-lab/actions/workflows/openshift-build.yml/badge.svg)](https://github.com/PaloAltoNetworks/redhat-openshift-lab/actions/workflows/openshift-build.yml)

Welcome to the self-service portal for provisioning ephemeral Red Hat OpenShift 4.x clusters on Google Cloud Platform (GCP). 

This repository uses GitHub Actions to automate the `openshift-install` process, allowing team members to spin up customized lab environments on demand. To prevent runaway cloud costs, **all clusters are strictly capped at a 72-hour lifespan** and will be automatically destroyed by a background Janitor process.

---

## Lab Architecture & Specs

To balance cloud cost efficiency with cluster stability, all clusters are deployed with a standardized footprint:

| Component | Count | Instance Type | Description |
| :--- | :--- | :--- | :--- |
| **Control Plane** | 3 | `e2-standard-4` | Standard HA Control Plane nodes running etcd. |
| **Worker Nodes** | 2 | `e2-standard-4` | **Hardcoded to 2 nodes.** Satisfies OpenShift router anti-affinity rules and prevents Ingress/OAuth operator resource starvation. |

---

## How to Provision a Cluster

You do not need to clone this repository or run any Terraform/CLI commands locally to get a cluster.

1. Navigate to the **[Actions](../../actions)** tab at the top of this repository.
2. On the left sidebar, click on **Provision OpenShift Lab**.
3. On the right side of the screen, click the **Run workflow** dropdown button.
4. Fill out the form:
   * **Cluster Name:** Provide a unique, identifiable name (e.g., `shrey-ocp-test-1`).
   * **Allowed IPs / CIDRs:** Provide your public IP or a comma-separated list of team IPs/subnets (e.g., `203.0.113.45, 198.51.100.0/24`). Find your IP at [ifconfig.me](https://ifconfig.me).
   * **Acknowledgment:** You must check the box acknowledging the 72-hour automated deletion policy to proceed.
5. Click the green **Run workflow** button. 

> **Note:** The installation process typically takes **45 to 50 minutes** to complete.

---

## Zero-Trust Security & Network Access

Clusters are built with a **Zero-Trust Default Architecture**. Unlike standard OpenShift installs that expose administrative endpoints to `0.0.0.0/0`, this repository automatically enforces firewall lockdowns immediately upon deployment.

### How Network Lockdown Works
At the end of the provisioning process, the pipeline automatically secures both external entry points using a hybrid network patch:
1. **API Server (`tcp:6443`):** Patched at the GCP Infrastructure layer using `gcloud` to lock static Load Balancer rules to your specified IP(s).
2. **Ingress Controller / Web Console (`tcp:80,443`):** Patched natively via OpenShift Operator CRDs (`oc patch ingresscontroller default`) with `scope: External` to dynamically constrain GCP Load Balancer target ranges.

---

## On-Demand Access Updates (JIT Workflow)

Because your cluster is locked down at launch, you **do not** need to manually lock it after build completion.

However, if your public IP changes (e.g., switching location, joining/leaving VPN) or you need to grant access to additional teammates later in the week, use the **Secure Cluster Access (JIT)** workflow:

1. Go to the **[Actions](../../actions)** tab.
2. Select **Secure Cluster Access** from the left sidebar.
3. Click **Run workflow**.
4. Enter your **Cluster Name** and your updated **Comma-Separated IPs/CIDRs** (e.g., `203.0.113.45, 198.51.100.12`).
5. Click **Run workflow**.

> Allow **2–3 minutes** after running JIT for GCP Load Balancers and OpenShift operators to sync and re-allow your traffic.

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

To manage cloud costs and resource limits, this repository enforces a strict **72-hour lifespan** on all OpenShift lab clusters.

* **Maximum Lifespan:** 72 Hours.
* **How it works:** When your cluster is built, its creation metadata is logged in a secure GCP bucket. Every 24 hours, the Janitor script checks this bucket. If your cluster is older than 72 hours, it will automatically trigger an `openshift-install destroy cluster` command.
* **Warning:** There is no backup of your lab data. Please ensure any important application manifests or configurations are committed to version control before the 72-hour window expires.

**Need a cluster deleted before 72 hours?**
If you have finished your testing early and want to free up resources, please request an early teardown:
1. Ping **Shrey Nilesh Raut** via Slack.
2. Provide the exact **Cluster Name** you want destroyed to request a `force` deletion for the created cluster.

---

## Currently in the works / shipping next:

* **Multi-IP Whitelisting**: Expanding the network patcher to accept comma-separated IP lists and CIDR subnets so teams can share cluster access or pair-program smoothly.
* **Custom OpenShift Versions**: Adding version selection upfront so engineers can target specific OCP releases (or stable channels) instead of defaulting to latest.

---
