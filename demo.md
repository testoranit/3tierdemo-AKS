

db micro services
db for 

different micros service..
pick micro-->contariexe them(usring docker files)-->K8Mainfest files-->Convert using heml-->deploy on AKS.


,\
u can do deply using K8s maifest directly to cluseter
so eg:- u hve 8 microservices so u need 16 yaml files(8 for each micro services, and 8 for service of each application),3 db's (mongo db 1 app yaml and service yaml)
(1 mysql db:- a app yaml and 1 service yaml, and 1 redis db=1 app and 1 service)=total 24 yamls
u can do it using kubectl apply/create

in real time org
u have diff env's dev stag prod
and u may have different parameter
eg:- dev env, each pod must not access more then 512 MB ram
but in Prod case could be different.
if u have 5 env's=24*5= yaml's so it would be super comples.
go to K8s foder in github
helm
1)charts.yaml:-metada of the charts if u have multiple versions.
2)templates.yaml= all yaml files(keep)
3)values.yaml=values that are dynamic as per env.


************

go to ur local clone git repo
u have code on ur local

also install azure cli on local.(also install kubectl aand helm)

az aks get-credentials --resource-group <rgname> --name <name of cluster>
rgname=ecomm-demo
name of cluseter=three-tier-cluster

![connect to AKS](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/4dcf3756-7201-4a86-8b3c-8ea00d62abec)


Now deploy this manifests using helms.
go to AKS/helm dir
kubectl create ns robot-shop
helm install robot-shop --namespace robot-shop .

![deploy pods using heml](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/389bab3e-644d-4aef-9822-247b078ad132)

![kubcetl get pods](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/4edbba48-c360-47b4-9808-1218e79df132)


kubcetl get storageclass
kubcetl get pods -n tobot-shop
kubect descibe pod redis-o -n robot-shop
kubectl get pvc -n robot-shop(disk created for redis,PV)
kubectl get svc -n robot-shop
(All pods and services are now running)

![service](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/62f58f5b-e460-403e-b06c-213bcefba8d1)

You now have LB service created u can access the robot-shop aap using this LB ip
![APp access using LB](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/0ee6ca09-2d95-468a-a2e2-46ed3ee38cd6)

BUt we here are using ingress

got to cluset on portal-->networking-->Application gateway Enable Inreess
in src code u have ingress.yaml
ingressClassName:azure application ateway
kubectl apply -f ingress.yaml -n robot-shop
kubectl get pods -A | grep ingress-app

If u don't see Application gateay Enable Ingress in Networking then
https://learn.microsoft.com/en-us/azure/application-gateway/tutorial-ingress-controller-add-on-existing
and then u can use inrees..


Gen:- check with dev tem on how to star the pp and include in in Dockerfile in cmd

MOnitoring AKS..

IMP:- Platform metrics for AKS cluster are gathered automatically with no extra cost
got to Overview of AKS-->Monitoring

![platform metrics](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/0ee8ec51-6f17-4f5e-bccc-41025c591027)


Access Container insights
Access Container insights in the Azure portal from Containers in the Monitor menu or directly from the selected AKS cluster by selecting Insights. The Azure Monitor menu gives you the global perspective of all the containers that are deployed and monitored. 

![before enabling container insights](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/dd6e8730-09ce-4bc9-93fa-f58b1c1f277c)

ABove pic shows no COntainer Insights are currently enabled.

Agent
Container insights and Managed Prometheus rely on a containerized Azure Monitor agent for Linux. This specialized agent collects performance and event data from all nodes in the cluster. The agent is deployed and registered with the specified workspaces during deployment. When you enable Container insights on a cluster, a Data collection rule (DCR) is created with the name MSCI-<cluster-region>-<cluster-name> that contains the definition of data that should be collected by Azure Monitor agent.
Is there support for collecting Kubernetes audit logs for ARO clusters? No. Container insights don't support collection of Kubernetes audit logs.
Is it possible for a single AKS cluster to use multiple Log Analytics workspaces in Container Insights? No. Container insights only accepts one Log Analytics Workspace in Container Insights for each AKS cluster.



https://learn.microsoft.com/en-us/azure/azure-monitor/containers/kubernetes-monitoring-enable?tabs=cli
Prerequisites
Permissions

**You require at least Contributor access to the cluster for onboarding.
**You require Monitoring Reader or Monitoring Contributor to view data after monitoring is enabled.

This article describes how to enable complete monitoring of your Kubernetes clusters using the following Azure Monitor features:
Managed Prometheus for metric collection
Container insights for log collection
Managed Grafana for visualization.

Using the Azure portal, you can enable all of the features at the same time. You can also enable them individually by using the Azure CLI, Azure Resource Manager template, Terraform, or Azure Policy. Each of these methods is described in this article.

!!
 Important

Kubernetes clusters generate a lot of log data, which can result in significant costs if you aren't selective about the logs that you collect. Before you enable monitoring for your cluster, see the following articles to ensure that your environment is optimized for cost and that you limit your log collection to only the data that you require:

Configure data collection and cost optimization in Container insights using data collection rule

*****************************************************************


****Supported clusters
Azure Kubernetes clusters (AKS)
Arc-enabled Kubernetes clusters


Managed Identity.

check:-
az aks show --resource-group ecomm-demo --name three-tier-cluster --query identityProfile

Managed Prometheus prerequisites

The cluster must use managed identity authentication.
The following resource providers must be registered in the subscription of the AKS cluster and the Azure Monitor workspace:
Microsoft.ContainerService
Microsoft.Insights
Microsoft.AlertsManagement
Microsoft.Monitor
The following resource providers must be registered in the subscription of the Grafana workspace subscription:
Microsoft.Dashboard

Feature	Workspace	Notes
Managed Prometheus	Azure Monitor workspace	Contributor permission is enough for enabling the addon to send data to the Azure Monitor workspace. You will need Owner level permission to link your Azure Monitor Workspace to view metrics in Azure Managed Grafana. This is required because the user executing the onboarding step, needs to be able to give the Azure Managed Grafana System Identity Monitoring Reader role on the Azure Monitor Workspace to query the metrics.
Container insights	Log Analytics workspace	You can attach an AKS cluster to a Log Analytics workspace in a different Azure subscription in the same Microsoft Entra tenant, but you must use the Azure CLI or an Azure Resource Manager template. You can't currently perform this configuration with the Azure portal.

If you're connecting an existing AKS cluster to a Log Analytics workspace in another subscription, the Microsoft.ContainerService resource provider must be registered in the subscription with the Log Analytics workspace. For more information, see Register resource provider.

For a list of the supported mapping pairs to use for the default workspace, see Region mappings supported by Container insights.
Managed Grafana	Azure Managed Grafana workspace	Link your Grafana workspace to your Azure Monitor workspace to make the Prometheus metrics collected from your cluster available to Grafana dashboards.


az aks show --resource-group ecomm-demo --name three-tier-cluster --query "identity.type"
CHeck the cluster Identity
az extension list --query "[?name=='k8s-extension']"
check k8s-extension.

and execute az aks update --enable-azure-monitor-metrics -n three-tier-cluster -g ecomm-demo

output:-
vm-admin@tomcat-vm:~$ az aks update --enable-azure-monitor-metrics -n three-tier-cluster -g ecomm-demo

(options:-


### Use default Azure Monitor workspace
az aks create/update --enable-azure-monitor-metrics -n <cluster-name> -g <cluster-resource-group>

### Use existing Azure Monitor workspace
az aks create/update --enable-azure-monitor-metrics -n <cluster-name> -g <cluster-resource-group> --azure-monitor-workspace-resource-id <workspace-name-resource-id>

### Use an existing Azure Monitor workspace and link with an existing Grafana workspace
az aks create/update --enable-azure-monitor-metrics -n <cluster-name> -g <cluster-resource-group> --azure-monitor-workspace-resource-id <azure-monitor-workspace-name-resource-id> --grafana-resource-id  <grafana-workspace-name-resource-id>

### Use optional parameters
az aks create/update --enable-azure-monitor-metrics -n <cluster-name> -g <cluster-resource-group> --ksm-metric-labels-allow-list "namespaces=[k8s-label-1,k8s-label-n]" --ksm-metric-annotations-allow-list "pods=[k8s-annotation-1,k8s-annotation-n]")


Using Azure Monitor Workspace (stores prometheus metrics) : /subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourceGroups/DefaultResourceGroup-westus2/providers/microsoft.monitor/accounts/DefaultAzureMonitorWorkspace-westus2
{
  "aadProfile": null,
  "addonProfiles": {
    "azureKeyvaultSecretsProvider": {
      "config": {
        "enableSecretRotation": "true"
      },
      "enabled": true,
      "identity": {
        "clientId": "b5c3d4b8-f57b-4e84-b039-4333a00495a3",
        "objectId": "c365b6f4-e0de-485d-857f-12eb5ee7d9f4",
        "resourceId": "/subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourcegroups/MC_ecomm-demo_three-tier-cluster_westus2/providers/Microsoft.ManagedIdentity/userAssignedIdentities/azurekeyvaultsecretsprovider-three-tier-cluster"
      }
    },
    "azurepolicy": {
      "config": null,
      "enabled": false,
      "identity": null
    }
  },
  "agentPoolProfiles": [
    {
      "availabilityZones": null,
      "capacityReservationGroupId": null,
      "count": 2,
      "creationData": null,
      "currentOrchestratorVersion": "1.28.5",
      "enableAutoScaling": true,
      "enableEncryptionAtHost": null,
      "enableFips": false,
      "enableNodePublicIp": false,
      "enableUltraSsd": null,
      "gpuInstanceProfile": null,
      "hostGroupId": null,
      "kubeletConfig": null,
      "kubeletDiskType": "OS",
      "linuxOsConfig": null,
      "maxCount": 5,
      "maxPods": 110,
      "minCount": 2,
      "mode": "System",
      "name": "agentpool",
      "networkProfile": null,
      "nodeImageVersion": "AKSUbuntu-2204gen2containerd-202403.25.0",
      "nodeLabels": null,
      "nodePublicIpPrefixId": null,
      "nodeTaints": null,
      "orchestratorVersion": "1.28.5",
      "osDiskSizeGb": 128,
      "osDiskType": "Managed",
      "osSku": "Ubuntu",
      "osType": "Linux",
      "podSubnetId": null,
      "powerState": {
        "code": "Running"
      },
      "provisioningState": "Succeeded",
      "proximityPlacementGroupId": null,
      "scaleDownMode": null,
      "scaleSetEvictionPolicy": null,
      "scaleSetPriority": null,
      "spotMaxPrice": null,
      "tags": null,
      "type": "VirtualMachineScaleSets",
      "upgradeSettings": {
        "drainTimeoutInMinutes": null,
        "maxSurge": "10%",
        "nodeSoakDurationInMinutes": null
      },
      "vmSize": "Standard_DS2_v2",
      "vnetSubnetId": null,
      "workloadRuntime": null
    }
  ],
  "apiServerAccessProfile": null,
  "autoScalerProfile": {
    "balanceSimilarNodeGroups": "false",
    "expander": "random",
    "maxEmptyBulkDelete": "10",
    "maxGracefulTerminationSec": "600",
    "maxNodeProvisionTime": "15m",
    "maxTotalUnreadyPercentage": "45",
    "newPodScaleUpDelay": "0s",
    "okTotalUnreadyCount": "3",
    "scaleDownDelayAfterAdd": "10m",
    "scaleDownDelayAfterDelete": "10s",
    "scaleDownDelayAfterFailure": "3m",
    "scaleDownUnneededTime": "10m",
    "scaleDownUnreadyTime": "20m",
    "scaleDownUtilizationThreshold": "0.5",
    "scanInterval": "10s",
    "skipNodesWithLocalStorage": "false",
    "skipNodesWithSystemPods": "true"
  },
  "autoUpgradeProfile": {
    "nodeOsUpgradeChannel": "NodeImage",
    "upgradeChannel": "patch"
  },
  "azureMonitorProfile": {
    "metrics": {
      "enabled": true,
      "kubeStateMetrics": {
        "metricAnnotationsAllowList": "",
        "metricLabelsAllowlist": ""
      }
    }
  },
  "azurePortalFqdn": "three-tier-cluster-dns-emvdxrid.portal.hcp.westus2.azmk8s.io",
  "currentKubernetesVersion": "1.28.5",
  "disableLocalAccounts": false,
  "diskEncryptionSetId": null,
  "dnsPrefix": "three-tier-cluster-dns",
  "enablePodSecurityPolicy": null,
  "enableRbac": true,
  "extendedLocation": null,
  "fqdn": "three-tier-cluster-dns-emvdxrid.hcp.westus2.azmk8s.io",
  "fqdnSubdomain": null,
  "httpProxyConfig": null,
  "id": "/subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourcegroups/ecomm-demo/providers/Microsoft.ContainerService/managedClusters/three-tier-cluster",
  "identity": {
    "delegatedResources": null,
    "principalId": "89d80f8d-365e-41fd-b9d7-c89dbc01ae85",
    "tenantId": "0d902a88-4027-4f87-bf95-c4c007c18cb5",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
  },
  "identityProfile": {
    "kubeletidentity": {
      "clientId": "7c195076-860b-450c-9d37-567add15f5db",
      "objectId": "826fd204-f10d-402d-b6e4-ae7e0f92a85c",
      "resourceId": "/subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourcegroups/MC_ecomm-demo_three-tier-cluster_westus2/providers/Microsoft.ManagedIdentity/userAssignedIdentities/three-tier-cluster-agentpool"
    }
  },
  "ingressProfile": {
    "webAppRouting": {
      "dnsZoneResourceIds": null,
      "enabled": true,
      "identity": {
        "clientId": "9a239ef2-fc03-427e-ba23-1d03cb3695cc",
        "objectId": "15ea8d62-be8b-439e-a74d-83de64ca38dc",
        "resourceId": "/subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourcegroups/MC_ecomm-demo_three-tier-cluster_westus2/providers/Microsoft.ManagedIdentity/userAssignedIdentities/webapprouting-three-tier-cluster"
      }
    }
  },
  "kubernetesVersion": "1.28.5",
  "linuxProfile": null,
  "location": "westus2",
  "maxAgentPools": 100,
  "name": "three-tier-cluster",
  "networkProfile": {
    "dnsServiceIp": "10.0.0.10",
    "ipFamilies": [
      "IPv4"
    ],
    "loadBalancerProfile": {
      "allocatedOutboundPorts": null,
      "backendPoolType": "nodeIPConfiguration",
      "effectiveOutboundIPs": [
        {
          "id": "/subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourceGroups/MC_ecomm-demo_three-tier-cluster_westus2/providers/Microsoft.Network/publicIPAddresses/5fcbaec0-264d-4ce6-b1f4-ad6743def6a9",
          "resourceGroup": "MC_ecomm-demo_three-tier-cluster_westus2"
        }
      ],
      "enableMultipleStandardLoadBalancers": null,
      "idleTimeoutInMinutes": null,
      "managedOutboundIPs": {
        "count": 1,
        "countIpv6": null
      },
      "outboundIPs": null,
      "outboundIpPrefixes": null
    },
    "loadBalancerSku": "Standard",
    "natGatewayProfile": null,
    "networkDataplane": "azure",
    "networkMode": null,
    "networkPlugin": "azure",
    "networkPluginMode": null,
    "networkPolicy": null,
    "outboundType": "loadBalancer",
    "podCidr": null,
    "podCidrs": null,
    "serviceCidr": "10.0.0.0/16",
    "serviceCidrs": [
      "10.0.0.0/16"
    ]
  },
  "nodeResourceGroup": "MC_ecomm-demo_three-tier-cluster_westus2",
  "oidcIssuerProfile": {
    "enabled": false,
    "issuerUrl": null
  },
  "podIdentityProfile": null,
  "powerState": {
    "code": "Running"
  },
  "privateFqdn": null,
  "privateLinkResources": null,
  "provisioningState": "Succeeded",
  "publicNetworkAccess": null,
  "resourceGroup": "ecomm-demo",
  "resourceUid": "661cf1d2d0847d0001e6fa05",
  "securityProfile": {
    "azureKeyVaultKms": null,
    "defender": null,
    "imageCleaner": null,
    "workloadIdentity": null
  },
  "serviceMeshProfile": null,
  "servicePrincipalProfile": {
    "clientId": "msi",
    "secret": null
  },
  "sku": {
    "name": "Base",
    "tier": "Free"
  },
  "storageProfile": {
    "blobCsiDriver": null,
    "diskCsiDriver": {
      "enabled": true
    },
    "fileCsiDriver": {
      "enabled": true
    },
    "snapshotController": {
      "enabled": true
    }
  },
  "supportPlan": "KubernetesOfficial",
  "systemData": null,
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters",
  "upgradeSettings": null,
  "windowsProfile": {
    "adminPassword": null,
    "adminUsername": "azureuser",
    "enableCsiProxy": true,
    "gmsaProfile": null,
    "licenseType": null
  },
  "workloadAutoScalerProfile": {
    "keda": null,
    "verticalPodAutoscaler": null
  }
}
vm-admin@tomcat-vm:~$





Enable COntainer Insights.

### Use default Log Analytics workspace
az aks enable-addons -a monitoring -n <cluster-name> -g <cluster-resource-group-name>

### Use existing Log Analytics workspace
az aks enable-addons -a monitoring -n three-tier-cluster -g ecomm-demo --workspace-resource-id /subscriptions/5b8ab9e9-05ae-4317-80d9-873b9c409693/resourceGroups/tomcat-iaas-12-2020/providers/Microsoft.OperationalInsights/workspaces/sampletest





Verify deployment
kubectl get ds ama-metrics-node --namespace=kube-system

Verify that the two ReplicaSets were deployed for Prometheus
kubectl get rs --namespace=kube-system

Container insights
Verify that the DaemonSets were deployed properly on the Linux node pools
User@aksuser:~$ kubectl get ds ama-logs --namespace=kube-system
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ama-logs   2         2         2         2            2           <none>          1d
Verify deployment of the Container insights solution
kubectl get deployment ama-logs-rs --namespace=kube-system

View configuration with CLI

Use the aks show command to find out whether the solution is enabled, the Log Analytics workspace resource ID, and summary information about the cluster

az aks show -g ecomm-demo -n three-tier-cluster


![after enabling container insights](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/f123dbdf-5c85-487e-94b1-80b7d3d45df5)

![Container Insighta](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/51bc27f5-ab5d-467e-9ce1-fb2ba642d043)

Promethus
https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/prometheus-workbooks#troubleshooting
Grafanas

https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/grafana-plugin


U should have Monitor read access to ur Azure Monitor workspace
go to Azure monitor workspace-->IAM-->assign read role.

![prometheus-explorer-menu](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/ee1be991-d821-469b-a459-da46bb7dfa17)

![platform metrics](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/f4eee80c-67cc-4fb1-a365-f5404f30d8bb)

![prometheus-query](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/102c68cd-4608-4817-b226-a350154e3324)

![prometheus-workspace-add-query](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/ab0832e2-8002-42a2-a9ef-1ef188635f31)

![prometheus-query-workbook](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/52104663-3984-4d71-b072-297aa1ef3c41)

Azure Managed Grafana includes an Azure Monitor data source plug-in. By default

Build a Grafana dashboard
Go to the Grafana home page and select New Dashboard.

In the new dashboard, select Graph. You can try other charting options, but this article uses Graph as an example.

A blank graph shows up on your dashboard. Select the panel title and select Edit to enter the details of the data you want to plot in this graph chart.

Run geqry in the grafana dashboard
avg(rate(node_cpu_seconds_total{mode="user"}[10m]))
![Grafana dashboard](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/0e017719-0009-45e8-92ed-01c5b10fe624)

**************Gen
Levels of Monitoring:-
CLuster level monitoring
Pod Level monitoring(resource utlilization of pods,application metrics,& metrics related to replication or autoscaling of the pod)

K8s Monitoring tool:- by defualt doesn't have monitoring
1)Promethus
2)Grafana
3)Jagear
4)Kubernetes dashboad
5)kalili
6)Kubewatch

![monitoring layers](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/ef3e580b-c1e2-4fed-abf5-1c40c57c8ccd)

![FLow of monitoring](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/31751a9e-bef0-4018-8d40-53679351d495)


Node level metrics:_ perfm of each nodes,avail,cpu consumption
Pod level metrics:- same as above
use heapster project as a reference for K8s monitoring.

vm-admin@tomcat-vm:~$ kubectl top node
NAME                                CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
aks-agentpool-20519794-vmss000004   217m         11%    3582Mi          78%
aks-agentpool-20519794-vmss000005   231m         12%    2934Mi          64%
vm-admin@tomcat-vm:~$

vm-admin@tomcat-vm:~$ kubectl top pod -n robot-shop
NAME                         CPU(cores)   MEMORY(bytes)
cart-64c54bd6d7-28q2n        1m           28Mi
catalogue-577d55dcc8-jdtqp   1m           34Mi
dispatch-886d77bc8-tqf6w     4m           24Mi
mongodb-7474db6fdf-sgz68     4m           161Mi
mysql-5fb849d788-hfvmz       1m           236Mi
payment-6cc6b544db-6vmnp     1m           36Mi

In docker u can use docker logs -f con_id to view the logs of container
In K8s
kubectl logs -f pod -n robot-shop


PromQL:-
Scenario:- Check Pods in Pending state:-
kube_pod_status_phase{namespace="<your_namespace>", phase="Pending"}



Metric server is installed in kubesystem namespace

get pods -n kube-system | grep metrics
ama-metrics-64664898b8-5bvlt                    2/2     Running   0            9h
ama-metrics-ksm-d9c6f475b-k497l                 1/1     Running   1 (9h ago)   9h
ama-metrics-node-lghmn                          2/2     Running   1 (9h ago)   9h
ama-metrics-node-z2twf                          2/2     Running   0            9h
ama-metrics-operator-targets-68df4f4784-87t5j   2/2     Running   2 (9h ago)   9h
metrics-server-5f476446c6-plbd6                 2/2     Running   0            23h
metrics-server-5f476446c6-tb7k5                 2/2     Running   0            23h
vm-admin@tomcat-vm:~$


**get compoent status ok K8S cluster
kubectl get componentstatus

If u want to collect data based on CPu
kubectl get pods --all-namespace --sort-by cpu > cpu_uti.txt

******
Logging
Application logs:_ to identify what is hapening inside app

Logs should have a sepearte storage and a lifecycle of independent of nodes,pods,or containers.This is called cluster-level logging.

![logging pipeline](https://github.com/testoranit/3tierdemo-AKS/assets/124513439/5f133126-3fdb-4db7-b56e-7169bd6ca38c)

Once logs are collected from Data source then use aggragtor like Fluentd and store it in logstorage like Elastic Search.
and Visualize using kibana/Grafana.

use this course for above notes:-
https://tcsglobal.udemy.com/course/certified-kubernetes-application-developer-training/learn/lecture/31046868#overview

**Loggig and debugging
If it's not Managed K8s and u have ur own K8s cluster below log components could be helpful.
API server is the acces point of the cluster (all calls go through it)
u can put log aggregator here.

2)scheduller:-element that determines where to run the containers.logging and debuging is necessary

3)etcd:- db (logging & debugging is also required here)

if u have done K8s instalaltion using kubeadm
u can execute kubectl logs above_component(etcd db) -n kube-system 

On-prem installation not done using kubeadm
then do,
journalctl -u kube-apiserver(in linux only)
or get logs from /var/log/kube-apiserver.log or /var/log/continer/*


****Logging using sidecar
Application generates logs and stores a specific loc, then u use sidecar to read these los.
Sidecar pattern:- pod with multiple con's where 1 pod(app) generates some logs to a location and other pods reads those logs from those loc)

gen:- kubectl logs pod-name -n robot-shop


****Application failures
Troubleshooting guide:-
to list all pods in all namespaces and check status=Running

kubectl get pods --all-namespaces

if u r getting problem in any of the pod
kubectl describe pod podname
if u status is Running then app inside that pod is running.
but
if status=pending then
Reason:_ not enough resources
so either delete existing pods or add a new worker node.

if status=waiting then
reason:- incorrect image name or image name that u specify doesn't present in ur repo.

******************Troubleshoot K8s Cluster failures
kubectl get nodes
kubectl describe node nod_name

complete cluster info
kubectl cluster-info or kubectl cluster-info dump

kubectl get component-status

Troubleshooting Service:-
in Kubeadm
Kubectl get pods -n kube-system

systemctl start kubelet
systemctl enable kubelet

systemctl start docker
systemctl enble docker

2)Manual Installaiton:-
systemctl deamon-reload
systemctl start kubelet
systemctl enable kubelet

systemctl start docker
systemctl enble docker

Troubleshooting Cluetr worker node
kubectl get nodes

Possible failure reasons:-
1)memory issue
2)network unvailaity
3)kubelet failure
4)CPU utlization
5)process failure

kubectl describe node node-name



