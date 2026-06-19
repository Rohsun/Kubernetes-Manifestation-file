SERVICE MESH - ISTIO NOTES


What is Service Mesh?

Service Mesh is a dedicated infrastructure layer that manages communication between microservices.

In microservices architecture, applications need:

- Service-to-service communication
- Load balancing
- Retry
- Timeout
- Security
- Encryption
- Metrics
- Distributed tracing


Without Service Mesh:

Frontend

 |

Backend

 |

Database


Application code handles:

- Routing
- Retry
- Security
- Monitoring


With Service Mesh:

Frontend

 |

Envoy Proxy

 |

Envoy Proxy

 |

Backend


Envoy proxy handles communication between services.



ISTIO ARCHITECTURE


Istio has two main parts:


1. Control Plane

2. Data Plane



CONTROL PLANE - Istiod


Istiod is the brain of Istio.


Responsibilities:

- Service discovery
- Certificate management
- Configuration distribution
- Traffic management
- Security policy management


Example:

When we create:

- VirtualService
- DestinationRule


Istiod reads those configurations and sends them to Envoy proxies.



DATA PLANE - Envoy Proxy


Envoy is a sidecar proxy injected into application pods.


Normal Pod:

Pod

 |

Application Container



After Istio injection:

Pod

 |

Application Container

 |

Envoy Proxy



Traffic flow:

Application

 |

Envoy Proxy

 |

Other Service


Envoy handles:

- Traffic routing
- Load balancing
- Retry
- Timeout
- Encryption
- Metrics



ISTIO CORE COMPONENTS



1. Istiod


Istiod manages the service mesh.


Functions:

- Pushes configuration to Envoy
- Manages certificates
- Controls traffic rules
- Handles service discovery



2. Envoy Proxy


Envoy runs beside application containers.


Responsibilities:

- Intercepts traffic
- Applies routing rules
- Provides security
- Collects telemetry



3. Istio Ingress Gateway


Gateway is the entry point for external traffic into the cluster.


Traffic flow:

User

 |

LoadBalancer

 |

Istio Gateway

 |

Service

 |

Pod


Used for:

- Exposing applications outside cluster
- TLS termination
- External traffic management



4. Istio Egress Gateway


Controls traffic leaving the cluster.


Example:

Application

 |

Envoy

 |

Egress Gateway

 |

External API


Used for:

- Controlling external communication
- Applying security rules



IMPORTANT ISTIO OBJECTS



1. Gateway


Gateway defines how traffic enters or exits the mesh.


Gateway is required only when external users access applications.


Example:

User

 |

Gateway

 |

Application



Gateway is NOT required for internal communication.


Example:

Frontend Pod

 |

Backend Service

 |

Backend Pod


No Gateway required.



2. VirtualService


VirtualService controls traffic routing.


It answers:

"Where should traffic go?"


Used for:

- Traffic splitting
- Canary deployment
- Blue-green deployment
- A/B testing


Example:


Incoming traffic


 |

VirtualService


 |

----------------

|

|

Version 1     Version 2


Traffic example:

80% traffic → v1

20% traffic → v2



3. DestinationRule


DestinationRule defines traffic policies after reaching a service.


It answers:

"How should traffic behave?"


Used for:

- Creating service versions
- Defining subsets
- Load balancing rules
- Connection policies


Example:


Service

 |

----------------

|

|

v1 pods      v2 pods



DestinationRule tells Istio:

These pods belong to v1

These pods belong to v2



VIRTUALSERVICE + DESTINATIONRULE


Both work together.


DestinationRule:

Defines versions/subsets.


Example:

backend-service

 |

----------------

|

|

v1 pods      v2 pods



VirtualService:

Controls traffic.


Example:

Request

 |

VirtualService


80% --------------> v1


20% --------------> v2



TRAFFIC FLOW WITH ISTIO


External traffic:


User

 |

Istio Gateway

 |

VirtualService

 |

DestinationRule

 |

Service

 |

Envoy Proxy

 |

Application Pod



Internal traffic:


Application Pod

 |

Envoy Proxy

 |

DestinationRule

 |

VirtualService

 |

Other Service



ISTIO FEATURES



Traffic Management:

- Routing
- Traffic splitting
- Retry
- Timeout
- Fault injection



Security:

- mTLS encryption
- Authentication
- Authorization



Observability:

- Metrics
- Logs
- Distributed tracing



KUBERNETES SERVICE vs ISTIO



Kubernetes Service:


Provides:

- Stable IP
- Service discovery
- Basic load balancing


Example:


Service

 |

Pod A

Pod B

Pod C



Istio:


Provides advanced traffic control.


Example:


Service

 |

VirtualService


90% traffic → Version 1

10% traffic → Version 2



MEMORY TRICK


Gateway

=

Entry door into cluster



VirtualService

=

Traffic controller

(Where traffic should go)



DestinationRule

=

Traffic policy

(How traffic should behave)



Envoy

=

Worker handling communication



Istiod

=

Brain managing everything



WHEN TO USE ISTIO?


Without Istio:

Application handles networking logic.


With Istio:

Application focuses on business logic.

Istio handles:

- Communication
- Security
- Routing
- Reliability
- Monitoring



LEARNING ORDER:


Deployment

↓

Service

↓

ConfigMap / Secret

↓

Probes

↓

HPA

↓

Storage

↓

Ingress

↓

Istio Service Mesh
