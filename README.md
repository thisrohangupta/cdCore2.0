# Introduction

## Changelog since Service V1 and Environment V1 Release

### Please read this before Upgrade!
- Service V1 and Environment V1 APIs are being deprecated on March 2023. 
- Existing pipelines referencing Service V1 and Environment V1 will run the risk of failure and no longer work after March 2023. 
- Harness will globally enable Service and Environments V2 APIs for all clients at the end of January 2023. T
- The forced change is needed to reduce the migration effort needed for users
- Harness has an automated tool to help migrate your services and environments from the v1 to v2 

## Changes by Kind
### Service 

- Service has a mandatory definition that needs to be configured via API or UI in order to use the service in a Pipeline

#### YAML Updates

```
service:
  name: nginx-canary
  identifier: nginxcanary
  
## SERVICE V2 UPDATE
## In V2, a Service Definition needs to be provided with the Service in order for it to be used in a pipeline 

  serviceDefinition:
    type: Kubernetes
    spec:
    
## SERVICE V2 UPDATE  
## You will need to provide a path to your Service Manifests (i.e. Kubernetes Manifests)

      manifests:
        - manifest:
            identifier: nginx
            type: K8sManifest
            spec:
              store:
                type: Github
                spec:
                  connectorRef: ProductManagementRohan
                  gitFetchType: Branch
                  paths:
                    - traffic-shifting-nginx/backend/deployment.yaml
                    - traffic-shifting-nginx/backend/service.yaml
                    - traffic-shifting-nginx/backend/nginx.yaml
                    - traffic-shifting-nginx/frontend/ui.yaml
                  repoName: Product-Management
                  branch: main
              valuesPaths:
                - traffic-shifting-nginx/values.yaml
              skipResourceVersioning: false
              
## SERVICE V2 UPDATE
## You will need to add an artifact if you want to pass an image tag in at pipeline runtime 
## this is also associated with the service configuration

      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - spec:
                connectorRef: public_dockerhub
                imagePath: library/nginx
                tag: <+input>
              identifier: nginx
              type: DockerRegistry
            
 ## SERVICE V2 UPDATE
 ## NEW CAPABILITY: We now have service variables that are mapped and managed with the service
 ## this can be overwritten when deploying the service to different environments
 
 
      variables:
        - name: canaryName
          type: String
          description: ""
          value: colors-canary
        - name: host
          type: String
          description: ""
          value: nginx-canary.harness.io
        - name: name
          type: String
          description: ""
          value: colors
        - name: stableName
          type: String
          description: ""
          value: colors-stable
  gitOpsEnabled: false

```

#### REST API UPDATES

- When creating a service via Harness REST API - Harness has exposed a new endpoint - https://apidocs.harness.io/tag/Services#operation/createServiceV2 

**Sample Payload Request**

```
{
  "identifier": "string",
  "orgIdentifier": "string",
  "projectIdentifier": "string",
  "name": "string",
  "description": "string",
  "tags": {
    "property1": "string",
    "property2": "string"
  },
  
## NEW PART OF THE SERVICE API PAYLOAD
## the YAML is optional is to provide via API for Service creation. 
## It's mandatory for the service to be used in a Pipeline 
## YAML is the Service Definition YAML passed as a string

  "yaml": "string" 
}
```

**Sample Payload Response**

```
{
  "status": "SUCCESS",
  "data": {
    "service": {
      "accountId": "string",
      "identifier": "string",
      "orgIdentifier": "string",
      "projectIdentifier": "string",
      "name": "string",
      "description": "string",
      "deleted": true,
      "tags": {
        "property1": "string",
        "property2": "string"
      },
      "yaml": "string"
    },
    "createdAt": 0,
    "lastModifiedAt": 0
  },
  "metaData": {},
  "correlationId": "string"
}
```

#### TERRAFOR PROVIDER

- The Terraform Provider Service Resource Endpoint HAS NOT CHANGED - https://registry.terraform.io/providers/harness/harness/latest/docs/resources/platform_service 
- The Service Resource Payload has a new field added for Service Creation - YAML 
- YAML is not mandatory for service object creation 
- When creating a service without YAML defined, it will create a skeleton service that cannot be used for immediate deployment
- The YAML field defines the actual definition of the Service so it can be used in a Pipeline for deployment

```
resource "harness_platform_service" "example" {
  identifier  = "identifier"
  name        = "name"
  description = "test"
  org_id      = "org_id"
  project_id  = "project_id"
  
## SERVICE V2 UPDATE
## We now take in a YAML that can define the service definition for a given Service
## It isn't mandatory for Service creation 
## It is mandatory for Service use in a pipeline

  yaml        = <<-EOT
                service:
                  name: name
                  identifier: identifier
                  serviceDefinition:
                    spec:
                      manifests:
                        - manifest:
                            identifier: manifest1
                            type: K8sManifest
                            spec:
                              store:
                                type: Github
                                spec:
                                  connectorRef: <+input>
                                  gitFetchType: Branch
                                  paths:
                                    - files1
                                  repoName: <+input>
                                  branch: master
                              skipResourceVersioning: false
                      configFiles:
                        - configFile:
                            identifier: configFile1
                            spec:
                              store:
                                type: Harness
                                spec:
                                  files:
                                    - <+org.description>
                      variables:
                        - name: var1
                          type: String
                          value: val1
                        - name: var2
                          type: String
                          value: val2
                    type: Kubernetes
                  gitOpsEnabled: false
              EOT
}
```


