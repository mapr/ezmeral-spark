# Helm Chart for MEP Spark History Server

[Spark History Server](https://spark.apache.org/docs/latest/monitoring.html#viewing-after-the-fact) provides a web UI for completed and running Spark applications. The supported storage backends are MaprFS and PersistentVolumeClaim (PVC)

## Installing the Chart

Note that this chart requires ECP tenant operator to be installed and Tenant CR applied in the tenant namespace.

### Install command
```sh
helm dependency update ./spark-hs-chart
helm install spark-hs ./spark-hs-chart -n sampletenant
```

This will create the helm chart in the `sampletenant` namespace.  This will create Spark history server with Spark 3.

#### Installing in a non DF Tenant
To install the helm chart in tenant type 'none' Namespace use the flag:  
`--set tenantIsUnsecure=true --set eventlogstorage.kind=pvc --set eventlogstorage.pvcname=spark-hs-pvc`  
Please note that if you are using an existing PVC, the pvc should exist in the same namespace.

##### Using custom keystore
To use a custom keystore, you'll need to create a secret with that keystore file in tenant namespace manually.
The secret should have keystore file stored under a particular key, e.g. "ssl_keystore".
Spark HS SSL configuration options can be passed to spark-hs in secure manner using 'sparkExtraConfigs' section, 
as shown in example below. Assuming that the secret name is "spark-ssl-secret", and the keystore key name in secret is 
"ssl_keystore", and passwords are "examplepass", update values.yaml like this:
```yaml
sparkSsl:
  useCustomKeystore: true
  sslSecretName: "spark-ssl-secret"
  secretMountPath: /var/spark

sparkExtraConfigs: |
  spark.ssl.historyServer.enabled           true
  spark.ssl.historyServer.keyStore          /var/spark/ssl_keystore
  spark.ssl.historyServer.keyStorePassword  examplepass
  spark.ssl.historyServer.keyPassword       examplepass
  spark.ssl.historyServer.protocol          TLSv1.2
  spark.ssl.historyServer.keyStoreType      PKCS12
```

##### Using S3 for storing logs
Alternatively you can create history server with existing s3 buckets for events log storage. To use this you can add the following flags to install command:
```
--set tenantIsUnsecure=true \
--set eventlogstorage.kind=s3 \
--set eventlogstorage.s3Endpoint=http://s3host:9000 \
--set eventlogstorage.s3path=s3a://bucket/folder \
--set eventlogstorage.s3AccessKey=AccessKey \
--set eventlogstorage.s3SecretKey=secretKey
```
If set, 's3AccessKey' and 's3SecretKey' configs will be passed to spark HS through a kubernetes secret.

Also, you can pass S3 credentials in secure way using "extra_configs" feature like this:
```yaml
sparkExtraConfigs: |
  spark.hadoop.fs.s3a.access.key [access_key]
  spark.hadoop.fs.s3a.secret.key [secret_key]
```

### Viewing the UI
After the chart is successfully installed, a message would be printed out to the console with details about how to access the UI.

### Disabling internal SSL for history server
By default, spark history SSL settings depend on the underlying DF security settings. In the DF is secure - SSL will be 
enabled implicitly. To disable SSL, put the following values to the values.yaml file:
```yaml
sparkExtraConfigs: |
  spark.ssl.historyServer.enabled           false

sparkSsl:
  enable: false
```

### Configuring spark HS with external OAuth proxy
Spark History Server can be configured to work behind OAuth proxy server. The proxy is responsible for user authentication,
and sends only the username down to the history server. For this setup, there are two things to be configured:
1. Spark HS should know that it runs behind the proxy
2. Authentication filter should be configured to use appropriate auth handler

Given that the proxy server runs at 'https://spark-hs.proxy.com' and the proxy sends username in 'X-USERNAME' header,
configuration in values.yaml will be:
```yaml
sparkExtraConfigs: |
  spark.ui.proxyRedirectUri	https://spark-hs.proxy.com
  spark.ui.filters org.apache.hadoop.security.authentication.server.AuthenticationFilter
  spark.org.apache.hadoop.security.authentication.server.AuthenticationFilter.param.type org.apache.spark.ui.filters.ProxyAuthenticationHandler
  spark.org.apache.hadoop.security.authentication.server.AuthenticationFilter.param.userheader X-USERNAME
```

### Configuring spark HS with ACLs
Spark History Server can be configured to work with ACLs. This feature needs to be enabled by spark.history.ui.acls.enable
Admins can see details of all jobs. Other users can see just details of their applications.
You also can change list of users who can see application details by setting spark.ui.view.acls when start Spark application.
```yaml
sparkExtraConfigs: |
  spark.history.ui.acls.enable true
  spark.history.ui.admin.acls mapr
```

## Uninstalling the Chart
Uninstall spark history server deployment named "spark-hs" from 'sampletenant' tenant
```
helm uninstall spark-hs -n sampletenant
```
Please note that this won't delete the PVC in case you are using an existing PVC. PVC will have to be manually deleted.
