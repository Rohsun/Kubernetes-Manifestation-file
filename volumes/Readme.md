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




