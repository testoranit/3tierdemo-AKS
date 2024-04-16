

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

and execute az aks update --enable-azure-monitor-metrics -n three-tier-cluster -g ecomm-demo

output:-
vm-admin@tomcat-vm:~$ az aks update --enable-azure-monitor-metrics -n three-tier-cluster -g ecomm-demo
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


