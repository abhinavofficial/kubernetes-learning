### 

Backup
```shell
#For the purpose of this test-drive, we have installed Kasten K10 and created a MySQL database k10demo in Kubernetes.

MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SHOW DATABASES LIKE 'k10demo'"
```

We will now create a policy to back up MySQL.
From the main Dashboard tab, click on the Policies card. There should be no policies visible at this point.

Click Create New Policy and:
Give the policy the name backup-mysql
Select Snapshot for the Action
Select Hourly for the Action Frequency
Leave the Snapshot Retention selection as-is
Select By Name for Select Applications and then, from the dropdown, select mysql
Leave all other settings as-is and select Create Policy

Running the Policy

The above policies will only run at the scheduled time (by default, at the top of the hour). To run the policies manually for the first time, click on Run Once on the Policies page. Confirm by clicking Run Policy and then go back to the main dashboard to view the job in action. Verify that the job has successfully completed.

Now that we have a MySQL backup, let's go simulate accidental data loss and then recover the system from that loss.
Causing Data Loss


```shell
#For the purposes of this test drive, we will simply drop the k10demo database we had created earlier.
MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "DROP DATABASE k10demo"

#Verify that the database has been deleted by running:
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SHOW DATABASES LIKE 'k10demo'"
```

Recovering Data

To recover the databases, go to the K10 dashboard, click Applications, and then select Restore on the MySQL card.

Click on a recent restore point and then select the Exported restore point (this is stored in the object storage system instead of as a non-durable snapshot on the storage system). In this case, we will select the default Application Name option to restore in place (Restore as "mysql"). Leave all other selections as-is, click on Restore, and confirm the action.

Return to the main dashboard to view the Restore job and verify that it completes successfully.


```shell
#To verify that our data was recovered, run the following command in the terminal to view the restored database:

MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SHOW DATABASES LIKE 'k10demo'"
```