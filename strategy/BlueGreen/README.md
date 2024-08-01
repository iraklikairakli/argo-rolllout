# Argo Rollouts BlueGreen Deployment Configuration

This document provides an overview of the configurable features available for BlueGreen deployments in Argo Rollouts. These features allow you to customize the deployment behavior according to your requirements.

## Configurable Features

Below are the optional fields that can be configured to modify the behavior of BlueGreen deployment:

### spec

#### strategy

**`blueGreen:`**

- **`autoPromotionEnabled`**: *(boolean)*  
  Specifies whether automatic promotion of the new version should occur.  
  - **true**: Automatically promote the new version after it becomes healthy.
  - **false**: Manual intervention required for promotion.

- **`autoPromotionSeconds`**: *int32*  
  The number of seconds to wait before automatically promoting the new version. Only applicable if `autoPromotionEnabled` is set to `true`.

- **`antiAffinity`**: *(object)*  
  Configures pod anti-affinity rules to prevent the new version and the previous version from being scheduled on the same nodes.

- **`previewService`**: *(string)*  
  The name of the Kubernetes service used to expose the preview version of the application.

- **`prePromotionAnalysis`**: *(object)*  
  Defines the analysis to be performed before promoting the new version. This can include custom checks or tests to ensure the new version is functioning correctly.

- **`postPromotionAnalysis`**: *(object)*  
  Defines the analysis to be performed after promoting the new version. This is useful for monitoring the application post-deployment.

- **`previewReplicaCount`**: *int32*  
  The number of replicas to run for the preview version of the application.

- **`scaleDownDelaySeconds`**: *int32*  
  The delay in seconds before scaling down the old version after the new version is promoted.

- **`scaleDownDelayRevisionLimit`**: *int32*  
  The maximum number of old revisions to retain during the scale-down process.

## irakli Configuration

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: irakli-rollout
spec:
  replicas: 5
  strategy:
    blueGreen:
      autoPromotionEnabled: true
      autoPromotionSeconds: 60
      antiAffinity:
        preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - my-app
              topologyKey: kubernetes.io/hostname
      previewService: "irakli-preview"
      prePromotionAnalysis:
        templates:
          - name: pre-promotion-checks
            templateName: "http-check"
      postPromotionAnalysis:
        templates:
          - name: post-promotion-monitoring
            templateName: "monitoring"
      previewReplicaCount: 2
      scaleDownDelaySeconds: 120
      scaleDownDelayRevisionLimit: 3
