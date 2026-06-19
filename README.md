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
