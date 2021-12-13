# RHACS Log4Shell mitigation example

A policy-based mitigation example for CVE-2021-44228 on Red Hat Advanced Cluster Security for Kubernetes.
The policy works on both build and deployment stages.

### How to use this example
Create a new policy on RHACS to detect the images and components affected by the CVE and 
trigger enforcement actions upon build and deploy.

To create the new policy import the policy configuration file `acs-log4shell-policy.json`. The policy 
is alreay configured to enforce both build and deployment stages.  


![Policy Import](images/import-policy.png)
  

Create a vulnerable deployment using image `quay.io/gbsalinetti/log4shell-vulnerable-app`, based on 
the [log4shell-vulnerable-app](https://github.com/christophetd/log4shell-vulnerable-app) example
by christophetd.
The application is a simple Spring Boot basic example that embeds log4j pacakge to print activity logs.

To create the deployment in the target cluster:

```
$ kubectl create -f log4shell-deployment.yaml
```

On RHACS this should produce an immediate action that prevents the deployment to be completed by scaling down
the expected replicas to 0.

```
$ oc describe deployment log4shell -n log4shell-rogue-ns
[...omitted output...]
Events:
  Type     Reason                Age   From                   Message
  ----     ------                ----  ----                   -------
  Warning  StackRox enforcement  5s    stackrox/sensor        Deployment violated StackRox policy "Log4Shell - CVE-2021-44228" and was scaled down
  Normal   ScalingReplicaSet     5s    deployment-controller  Scaled up replica set log4shell-f6d896fdd to 3
  Normal   ScalingReplicaSet     5s    deployment-controller  Scaled down replica set log4shell-f6d896fdd to 0
```

Looking at the RHACS console we can find detailed informations about the discovered vulnerability:

![CVE Details](images/cve-details.png)

**IMPORTANT**: The policy only affects new deployments and builds. Pre-existing deployments are not eradicated by default and should be mitigated by 
upgrading the underlying image or applying a proper mitigation like disabling Jndi lookup.

