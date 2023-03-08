# Gloo Mesh And Istio Egress Gateway Functionality

# Motivation

In this example, two clusters exist in separate networking regions but need access to the same service with a set of certificate information. In order to access this service from the remote cluster, traffic must be intercepted, sent through the east/west gateway and leave through the egress gateway. Ideally mTLS will be maintained throughout the entire journey as well.

# Example Architecture

![Diagram](./diagram.svg)

# Setup

- Gloo Mesh 2.2.4
- Istio 1.16
- 3 Workspaces
  - ops
  - app1
  - app2
- Workspace settings with federation and service isolation disabled
- Root Trust Policy for clusters

# Resources

- MGMT
  - external-endpoint.yaml
  - external-service.yaml
  - external-service-virtual-destination.yaml
  - egress-gateway-virtual-destination.yaml
  - egress-gateway-virtual-gateway.yaml
  - east-west-gateway-virtual-gateway.yaml
  - route-table-from-mesh-to-egress.yaml
  - route-table-from-egress-to-egress.yaml
  - route-table-from-egress-to-external-service.yaml
- Local
  - egress-istio-operator.yaml
  - destination-rule-source-namespace.yaml
  - destination-rule-gateway-namespace.yaml
  - tls-origination-destination-rule.yaml
  - peerauthentication.yaml
- Remote
  - destination-rule-source-namespace.yaml
  - peerauthentication.yaml

# Proof of Function

Assuming all of the resources have been deployed appropriately, then a request to httpbin.external from the `app1` or `app2` workspace like `curl httpbin.external` should result in log messages on the egress gateway and a response code of `200`.

To prove egress TLS origination, simply replace `SIMPLE` in `tls-origination-destination-rule.yaml` with `DISABLE` and perform a request as stated above.
The result should contain this message: `The plain HTTP request was sent to HTTPS port`

# Notes

- External Endpoint is required here to allow for the creation of a Virtual Destination (usually not needed)
- Virtual Destinations *must* have the clientMode specified as tlsTermination
- The virtual gateways are using selectors on istio gateways that are automatically created. (there is no corresponding IOP)
- East West gateways have an additional port 443, for tlsTermination
- The SNI field on destination rules does limit the defined path per resource. (one gateway per SNI)
- mTLS and egress TLS origination confirmed working
