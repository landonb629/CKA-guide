# Application lifecycle mgmt

Rolling updates and Rollbacks 

Deployment Strategies 
- Destroy & Recreate: if you have 3 versions of your app, destroy the 3 existing and create the new versions of the application
    - this is not the default deployment stragety 
    - this has downtime associated with it 

- Rolling update: each replica is taken down and replaced with the new version 
    - this is default strategy 
    - this doesn't have downtime associated 

How do you trigger a deployment?
- make a change to the existing deployment manifest, run ``` kubectl apply -f $FILE.yml ``` 

How can you check the status and details of your deployments?

checking the status
```  ```

checking the deployment events 
``` k describe deployment $DEPLOYMENTNAME ```

what happens under the hood when you trigger an upgrade?
- you deploy the first deployment
- you update a trigger and a new replica set will be deployed along side the old one 
- your pods will start getting taken down in the old RS and brought up in the new RS 


Rolling back a deployment 

```k rollback deployment $DeploymentName ```
