# Helm Chart for Tenant Operator and Validator

### Installing the Chart

#### Install command
```shell
helm install tenant-operator tenant-operator/ -n <namespace> --create-namespace
```

The chart installation will create Tenant operator and tenant validator webhook.
All the resources defined in the chart are created in <namespace> namespace.

## Uninstalling the Chart
```shell
helm delete tenant-operator -n <namespace>
```