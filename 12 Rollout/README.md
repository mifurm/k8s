<br><br>
<br><br>
<br><br>

# Rollout

## LAB Overview

#### In this lab you will work with Deployments. You will create a Deployment, update it, examine its history and will do a rollback.

## Task 1: Create a Deployment to rollout a ReplicaSet.

1. Create new file by typing ``nano depl.yaml``.
2. Download [manifest file](./files/depl.yaml) and paste its content into editor.
3. Save changes by pressing *CTRL+O* and *CTRL-X*.
4. Type ``kubectl apply -f depl.yaml`` and press enter.

## Task 2: Examinig the deplyment

1. Run ``kubectl get deployments`` to check if the Deployment was created.
![img](./img/deployment1.png)

2. You can also check rolout status of the Deployment if it's not ready:
``
kubectl rollout status deployment nginx-deployment
``
and run ``kubectl get deployments`` once again a few seconds later.
To see the ReplicaSet (rs) created by the Deployment, run ``kubectl get rs``.
![img](./img/deployment-rs.png)
3. You can also see the labels automatically created for each Pod:
``
kubectl get pods --show-labels
``
![img](./img/deployment-labels.png)
4. Select one of your Pods and create a proxy to it:
``
kubectl port-forward <-YOUR-POD-NAME-> 8080:80
``
![img](./img/deployment-proxy.png)
5. Connect to your *nginx* using any browser of your choice.
![img](./img/deployment-version1.png)
<br><br>
You can also use *curl* and check the response headers: ``curl -I -X GET http://localhost:8080``. You need to use different terminal window.
![img](./img/version1-headers.png)

As you can see, you're using nginx 1.7.9

Exit the proxy by pressing **CTRL+C**.

## Task 3: Updating the deployment

1. Still in the terminal run the following command:

``
kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1
``

2. Check the satus of the Deployment: 

``kubectl rollout status deployment.v1.apps/nginx-deployment
``

3. Get the list of ReplicaSets:

``
kubectl get rs
``

You can both old and new Replica Set created:
![img](./img/deployment-rs2.png)


4. Create the proxy to one of your Pod again: ``kubectl port-forward <-YOUR-POD-NAME-> 8080:80``.
Check Pod names again. you have new Pods in your Deployment now.
5. Use the **curl** once again and check the headers of the response:
``
curl -I -X GET http://localhost:8080
``
You should have now *nginx* ver. 1.9.1 in the headers.
![img](./img/headers2.png)
6. Open the manifest file by running:

``
nano depl.yaml
``

and edit the file:
* set replicas to: 5
* set image to: nginx:1.17.3
* to deployment's **spec** add:
```
strategy:
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
  type: RollingUpdate
```
* to deployment's **metadata** add

```
annotations:
  kubernetes.io/change-cause: "Image change"
```
If you're not quite qure how to do it, you can use [depl2.yaml file](depl2.yaml)

1. Update the Deployment using ``kubectl apply -f depl.yaml`` or ``kubectl apply -f depl2.yaml`` command.

After a while you'll have 5 pods running inside your Deployment.
![img](./img/pods2.png)

## Task 4: Managing Rollout History

1. Get the history of the Deployment by running:

``
kubectl rollout history deployment nginx-deployment
``
![img](./img/h1.png)

2. If you are interested in more details about a particular revision, you can add the *--revision* flag to view details about that specific revision:

``kubectl rollout history deployment nginx-deployment --revision=2``

![img](./img/det.png)

3. Undo the last rollout by running:

``
kubectl rollout undo deployments nginx-deployment
``

4. Check the history of Deployments

``
kubectl rollout history deployment nginx-deployment
``
![img](./img/h1.png)

As you notices, there is no longer Deployment 2, and Deployment 4 was added. Now we have revisions 1,3 and 4.

5. Rollback to revision 3 by running

``
kubectl rollout undo deployments nginx-deployment --to-revision=3
``
and check the history: ``kubectl rollout history deployment nginx-deployment``
![img](./img/hi3.png)

6. Please delete the Deployment:

``
kubectl delete deployment nginx-deployment
``

## END LAB