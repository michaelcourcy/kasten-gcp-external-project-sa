# Goal 

Show that you can deploy Kasten on GCP using a Google Service Account (GSA) wich is defined in another project than the project that hold your GKE cluster.

Kasten supports GKE by using a Google Service Account (GSA) that belongs to the same project than the google service account used by GKE cluster but we also support GSA that belongs to other project 

# Prequisite 

You need a GKE cluster running on the GKE cluster project's we'll call this project GKE_PROJECT.

You need another project for the management od the GSA we'll call this project MANAGEMENT_PROJECT

for instance 
```
GKE_PROJECT=rich-access-174020
MANAGEMENT_PROJECT=k10-gke-testdrive-180621
MANAGEMENT_SA=kasten
```

Create the account in the management project 
```
gcloud config set project $MANAGEMENT_PROJECT
gcloud iam service-accounts create $MANAGEMENT_SA
# grab the key locally 
gcloud iam service-accounts keys create --iam-account=$MANAGEMENT_SA@$MANAGEMENT_PROJECT.iam.gserviceaccount.com $MANAGEMENT_SA@$MANAGEMENT_PROJECT-sa-key.json
```

Give to the MANAGEMENT_SA the role compute.storageAdmin on GKE_PROJECT.

```
gcloud config set project $GKE_PROJECT
gcloud projects add-iam-policy-binding $GKE_PROJECT --member serviceAccount:$MANAGEMENT_SA@$MANAGEMENT_PROJECT.iam.gserviceaccount.com --role roles/compute.storageAdmin
```

control : 

```
gcloud config set project $GKE_PROJECT
gcloud projects get-iam-policy $GKE_PROJECT  \
--flatten="bindings[].members" \
--format='table(bindings.role)' \
--filter="bindings.members:$MANAGEMENT_SA@$MANAGEMENT_PROJECT.iam.gserviceaccount.com"
```

You should get this output 

```
ROLE
roles/compute.storageAdmin
```

# Install k10 with the key from the management cluster 

Login to your cluster 
```
gcloud container clusters get-credentials <cluster-name> --zone=<zone-name>
```

```
kubectl create ns kasten-io
helm install k10 kasten/k10 --namespace=kasten-io 
```

Access the dashboard 
```
kubectl --namespace kasten-io port-forward service/gateway 8080:8000
```

Note : Your installation may be more complex, refer to the kasten documentation

The Kasten dashboard will be available at: `http://127.0.0.1:8080/k10/#/`

Try to add the GCP infra profile 
- The gcp project is $GKE_PROJECT
- The google service account should be pasted from the $MANAGEMENT_SA@$MANAGEMENT_PROJECT-sa-key.json file