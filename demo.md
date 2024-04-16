

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




