# Kubernetes Notes


# Namespace

Create namespace

kubectl create namespace my-space

Namespace → Isolates Kubernetes resources.


Get namespaces

kubectl get ns


Apply resource in namespace

kubectl apply -f deployment.yml -n my-space


Delete all resources in namespace

kubectl delete all --all -n my-space

all → Deletes pods, services, deployments, replicasets etc.



# Pods

Get pods

kubectl get pods


Get pods from namespace

kubectl get pods -n my-space


Describe pod

kubectl describe pod <pod-name>

describe → Shows pod details, events and errors.


Enter inside pod

kubectl exec -it <pod-name> -- sh

exec → Execute commands inside container.


View pod logs

kubectl logs <pod-name>

logs → Shows application/container logs.



# Deployment

Apply deployment

kubectl apply -f deployment.yml

Deployment → Manages pods, replicas and application lifecycle.


Get deployments

kubectl get deployment


Scale deployment manually

kubectl scale deployment my-deployment --replicas=5

scale → Increase or decrease number of pods.


Delete deployment

kubectl delete deployment my-deployment



# Service

Get services

kubectl get svc

Service → Provides stable networking and access to pods.


Describe service

kubectl describe svc my-service


Check service endpoints

kubectl get endpoints my-service

Endpoint → Shows which pods are receiving traffic.


Port forward service

kubectl port-forward svc/my-service 8080:80

port-forward → Access service locally.



# NodePort

Service type:

type: NodePort


Get NodePort

kubectl get svc


Example:

80:31662


Access:

localhost:31662


NodePort → Exposes service outside Kubernetes cluster.



# ConfigMap

Create ConfigMap

kubectl apply -f config.yml


ConfigMap → Stores non-sensitive configuration.


Get ConfigMaps

kubectl get cm


Describe ConfigMap

kubectl describe cm my-config


Example:

data:

 app_env: dev

 app_name: hotstar


Pod receives:

APP_ENV=dev

APP_NAME=hotstar



# Secret

Create Secret

kubectl apply -f secret.yml


Secret → Stores sensitive information.

Examples:

Passwords
Tokens
API keys
Credentials


Get secrets

kubectl get secret


Describe secret

kubectl describe secret my-secret


Decode secret

echo YWRtaW4= | base64 -d


base64 → Encoding, not encryption.



# Readiness Probe


Example:

readinessProbe:

  httpGet:

    path: /

    port: 80


  initialDelaySeconds: 5

  periodSeconds: 10



Meaning:

Wait 5 seconds after container starts

Then check health every 10 seconds


Readiness → Controls traffic.


If readiness fails:

Pod removed from Service endpoint.



# Liveness Probe


Example:

livenessProbe:

  httpGet:

    path: /

    port: 80



Liveness → Checks application health.


If liveness fails:

Container restarted.



# Resource Requests and Limits


Example:

resources:


 requests:

   cpu: "250m"

   memory: "100Mi"


 limits:

   cpu: "500m"

   memory: "200Mi"



Requests:

Minimum guaranteed resources.

Used by scheduler to place pods.


Limits:

Maximum allowed resources.

Controls container resource usage.



# HPA (Horizontal Pod Autoscaler)


Apply HPA

kubectl apply -f horizontalscale.yml


HPA → Automatically scales pods based on CPU/memory usage.


Get HPA

kubectl get hpa


Check metrics

kubectl top pods


Metrics Server → Provides CPU and memory usage data.



# Debugging Commands


Get all resources

kubectl get all


Get events

kubectl get events


Check generated YAML

kubectl get deployment my-deployment -o yaml


Delete resource using YAML

kubectl delete -f deployment.yml



# Apply Order (Real Project)


Namespace

     |

ConfigMap

     |

Secret

     |

Deployment

     |

Service

     |

HPA



# Traffic Flow


User

 |

 v

Service

 |

 +------------+

 |            |

 v            v

Pod          Pod

 |

 v

Container


=======================================================================================================================================================================================================================================================================================================================================================================================
# Kubernetes Storage (PV / PVC / StorageClass) Notes


==================================================
Why do we need PV and PVC?
==================================================

Containers are temporary.

If a Pod is deleted:

Pod
 |
 |
Container storage
 |
 |
Data deleted


To keep data permanently, Kubernetes uses:

PV + PVC



Flow:

Application Pod

        |

       PVC

        |

       PV

        |

Actual Storage



==================================================
Persistent Volume (PV)
==================================================


PV = Actual storage available in Kubernetes.


PV contains details like:

- Storage size
- Access mode
- Storage location
- Reclaim policy


Example:

PV says:

"I have 10GB storage available"




PV can be created in two ways:


1. Static Provisioning

2. Dynamic Provisioning



==================================================
1. Static Provisioning
==================================================


Static means:

We manually create the storage first.


Flow:


Admin creates PV

        |

Admin creates PVC

        |

Pod uses PVC



Example:


Step 1:

Create PV manually


PV contains:

- Storage size
- Where storage exists
- Type of storage



Step 2:

Create PVC


PVC says:

"I need 5GB storage"


Kubernetes finds matching PV and binds them.



Flow:


PV

 |

PVC

 |

Pod



==================================================
Static Provisioning with hostPath
==================================================


Example:

PV uses:


hostPath:

/data/my-volume



Meaning:


Use storage from Kubernetes node filesystem.



Flow:


Node-1


/data/my-volume

        |

        |

       PV

        |

        |

       PVC

        |

        |

       Pod



Important:

hostPath storage belongs to that node only.



Example:


Cluster:


Node-1              Node-2


/data/my-volume


(PV exists here)


Pod running on Node-1


Data available



If Pod moves:


Node-1

Pod removed



Node-2

New Pod starts



Node-2 does not have:

/data/my-volume data



So:

hostPath = node specific storage



Used for:

- Learning
- Testing
- Single node clusters



Not recommended for:

- Production multi-node clusters



==================================================
Static Provisioning with External Storage
==================================================


Static does NOT mean only hostPath.



You can use external storage also:


Example:


AWS EBS

NFS

Azure Disk

GCP Persistent Disk



But the difference is:

You manually create PV.


Example:


External Storage exists:

AWS EBS Volume


        |

        |

You create PV pointing to it


        |

        |

Create PVC


        |

        |

Pod uses storage



So static always requires:


PV creation manually

+

PVC creation manually



==================================================
Dynamic Provisioning
==================================================


Dynamic means:

Kubernetes automatically creates PV.


You don't create PV manually.



Flow:


StorageClass created once

        |

Application creates PVC

        |

StorageClass creates PV automatically

        |

Pod uses PVC



Example:


Cluster Admin creates:


StorageClass


Example:

gp3



Later developer creates:


PVC


Request:

"I need 20GB storage"



Kubernetes:


StorageClass

        |

Creates PV

        |

Creates actual storage



Flow:


PVC

 |

StorageClass

 |

PV

 |

Storage



==================================================
StorageClass
==================================================


StorageClass = Storage template



It defines:

- Which storage system to use
- Which driver/provisioner creates storage
- How volumes are created



Created usually one time by:

Platform / Kubernetes admin



Example:


AWS:


StorageClass

 |

EBS CSI Driver

 |

AWS EBS Volume



Azure:


StorageClass

 |

Azure Disk Driver

 |

Azure Disk



Docker Desktop:


StorageClass

 |

hostpath

 |

Node filesystem



Applications reuse the same StorageClass.



==================================================
Static vs Dynamic Difference
==================================================


STATIC:


Admin creates:


PV

 |

PVC

 |

Pod



You manage storage details.






DYNAMIC:


Admin creates:


StorageClass


Developer creates:


PVC


Kubernetes creates:


PV automatically



==================================================
PVC (Persistent Volume Claim)
==================================================


PVC = Request for storage.


Application does not directly use PV.



Application says:


"I need 5GB storage"



PVC requests it.



PVC connects:


PVC ---> PV



PVC contains:


- Required storage size
- Access mode
- StorageClass (dynamic)



==================================================
Access Modes
==================================================


Defines how storage can be accessed.



ReadWriteOnce (RWO)


Meaning:

One node can read/write.



Example:

AWS EBS



----------------------------------


ReadOnlyMany (ROX)


Multiple nodes can read.



----------------------------------


ReadWriteMany (RWX)


Multiple nodes can read/write.



Example:

NFS

EFS



==================================================
Reclaim Policy
==================================================


Reclaim policy decides:

"What happens to storage after PVC deletion?"



There are mainly two important policies:



1. Retain
==================================================


Example:

PV:

ReclaimPolicy: Retain



Scenario:


Application running


PV

 |

PVC

 |

Pod



Now delete PVC:



PVC deleted



What happens?


PV stays

Data stays



PV status:


Released



Meaning:

"Kubernetes removed the connection,
but storage is still there"



Admin must manually clean it.



Used for:


- Databases
- Important data



Example:

MySQL data

Customer files



----------------------------------


2. Delete
==================================================


Example:

PV:

ReclaimPolicy: Delete



Scenario:


PVC deleted



What happens?


PVC deleted

        |

PV deleted

        |

Actual storage deleted



Used for:


Temporary data

Testing workloads



==================================================
Dynamic StorageClass Reclaim Policy
==================================================


In dynamic provisioning:

StorageClass usually decides default reclaim policy.



Example:


StorageClass

        |

Creates PV

        |

PV gets reclaim policy



Common:

Delete



Meaning:

Application removed

        |

PVC deleted

        |

PV deleted

        |

Cloud disk removed



==================================================
Important Difference
==================================================


Dynamic does NOT mean:

"Highly available"



Dynamic means:

"PV creation is automated"



Example:


Docker Desktop:


Dynamic

+

hostpath


Still:

Single node storage



Production:


Dynamic

+

AWS EBS/EFS


Can provide better availability



==================================================
Real Production Flow
==================================================


Platform Team:


Install storage driver


Create StorageClass



Example:


gp3


Application Team:


Create PVC


Deployment mounts PVC



Kubernetes:


Creates PV automatically



Final flow:


Application

 |

PVC

 |

StorageClass

 |

PV

 |

External Storage



==================================================
Commands
==================================================


Check PV:


kubectl get pv



Check PVC:


kubectl get pvc



Check StorageClass:


kubectl get storageclass



Detailed information:


kubectl describe pv <name>


kubectl describe pvc <name>


kubectl describe storageclass <name>



==================================================
Remember in one line
==================================================


Static:

"I create PV first, then PVC binds to it"


Dynamic:

"I create StorageClass once, request PVC, Kubernetes creates PV"


PV:

"Actual storage"


PVC:

"Request for storage"


StorageClass:

"Template to automatically create storage"


Reclaim Policy:

"What happens to storage after PVC deletion"





========================================================================================================================================================================================================================================================
# Important Commands


kubectl get pods

kubectl get svc

kubectl get deployment

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl exec -it <pod-name> -- sh

kubectl apply -f file.yml

kubectl delete -f file.yml

kubectl get all

kubectl get events
