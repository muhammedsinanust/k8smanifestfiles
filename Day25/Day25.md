Day25
-----

Ingress
-------
protocol-aware configuration mechanism, that understands web concepts like URIs, hostnames, paths.
Ingress lets you map traffic to different backends based on rules you define via the Kubernetes API.

Ingress may provide load balancing, SSL termination and name-based virtual hosting.

The Ingress API has been frozen.
The Kubernetes project has no plans to remove Ingress from Kubernetes.
The Ingress API is no longer being developed, and will have no further changes or updates made to it.

Service - identifies a set of Pods using label selectors.


Ingress - exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

https://kubernetes.io/docs/images/ingress.svg

Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

Ingress controller - responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.