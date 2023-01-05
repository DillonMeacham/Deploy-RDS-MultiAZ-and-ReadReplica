<h1>Deploy Amazon RDS Multi-AZ and Read Replica</h1>

<h2>Description</h2>
In this project, we will work with Relational Database Service (RDS) in AWS. Multi-AZ and read replicas serve different purposes with RDS. Multi-AZ is strictly for failover, as the standby instances cannot be read from by an application. Read replicas are used for improved performance and migrations. With read replicas, you can write to the primary database and read from the read replica. Since a read replica can be promoted to be the primary database, it makes for a great tool in disaster recovery and migrations.

<br />
<br />

<img src="https://imgur.com/neLkCh0.png" height="40%" width="40%" />


The diagram explains what we will be doing in this project. We are going to start out with one Amazon RDS instance in one AZ (Availability Zone). We will then enable Multi-AZ and Backups, creating a primary instance in AZ1 and a secondary instance in AZ2. We are then going to simulate failure by rebooting the primary instance, which will make the secondary instance in AZ2 the new primary and after the previous primary is done rebooting, it will now be the new secondary instance. We will then create and promote a Read Replica and simulate a failure by our primary instance going offline. Finally we will update the Route 53 record to point to the newly created Read Replica. 


<img src="https://imgur.com/ptkgLrF.png" height="85%" width="85%" />


<h2>Environments Used </h2>

- <b>AWS Management Console</b>
- <b>RDS</b>
- <b>Route 53</b>

<h2>Project walk-through:</h2>

I will be using a pre-loaded environment with a WordPress instance as the primary database. 
If we navigate to RDS >> Databases, we can scroll all the way to the right of the instance to see that Multi-AZ for our primary RDS database is turned 'Off'. This is the first thing we want to enable. 

<img src="https://imgur.com/WhLOh2K.png" height="70%" width="70%" />
<img src="https://imgur.com/GYj2kWG.png" height="70%" width="70%" />


Click on the radio button for out WordPress database and click 'modify'.

<img src="https://imgur.com/gaiTM7W.png" height="70%" width="70%" />

Under the 'Availability & Durability' section, we want to select 'Create a Standby Instance', this will create a standby instance in another availability zone. Click next, review changes, and confirm. 

<img src="https://imgur.com/3BJ5SJf.png" height="70%" width="70%" />
<img src="https://imgur.com/idSOyIz.png" height="70%" width="70%" />


Now we will time travel 6 minutes, to see that our Multi-AZ deployment is showing Available and  has been modified successfully. 

<img src="https://imgur.com/eFq9d6R.png" height="70%" width="70%" />
<img src="https://imgur.com/kufeSYl.png" height="70%" width="70%" />

Now let's simulate the Multi-AZ redundancy we just set up by rebooting our primary instance and letting our secondary instance takeover. We'll select our primary instance, click on 'Actions', and then 'Reboot', and also 'Reboot with Failover'. The goal here is to ensure our Wordpress site is still running while the reboot is in progress, because of our Multi-AZ deployment, we can confirm in the Event logs that this was successful. 

<img src="https://imgur.com/JsxDOnY.png" height="70%" width="70%" />
<img src="https://imgur.com/dwssLQ7.png" height="70%" width="70%" />
<img src="https://imgur.com/538z95r.png" height="70%" width="70%" />

Now let's create a Read Replicate for this database by selecting our database instance, clicking 'Actions' and 'Create Read replica'. We can leave most of the settings as default but the one thing we want to change is the identifier, we will change this to "Wordpress-RR" and click 'Create Read Replica' at the bottom. This will create a Read Replica of our database and keep our site available during this change because of Asynchronization. Meaning this replication is occurring after the data has already been written to the primary storage. So realistically in a production environment, we might only see CPU utilization spike a little bit during this change. 


<img src="https://imgur.com/RPqBdXk.png" height="70%" width="70%" />
<img src="https://imgur.com/tSjuMtW.png" height="70%" width="70%" />
<img src="https://imgur.com/EWZIEpn.png" height="70%" width="70%" />
<img src="https://imgur.com/Aft3I4a.png" height="70%" width="70%" />

Now the we have an available replica, in the case where our Wordpress Database has gone offline, all we have to do is promote the read replica to a standard instance and update our DNS records. This will allow us to quickly recover form a disaster situation. To do this we will select our read replica, click on 'Promote' under 'Actions', and confirm changes. After some time, we can now see that our Role for our read replica has changed to 'Instance'.

<img src="https://imgur.com/gLzLd9K.png" height="70%" width="70%" />
<img src="https://imgur.com/yb6CwCy.png" height="70%" width="70%" />
<img src="https://imgur.com/SGPp6ao.png" height="70%" width="70%" />

Now lets update our DNS records in Route 53. If we click on the read replica, we can copy the endpoint and navigate to Route 53 >> Hosted zones and click on 'mydomain.local' and we can edit the record value and paste our endpoint link into the value box and click 'Save'.

<img src="https://imgur.com/E2dK2Bi.png" height="70%" width="70%" />
<img src="https://imgur.com/eqQDpbO.png" height="70%" width="70%" />

That concludes this project and why this is a great solution for a disaster recovery situation to ensure high availability for your databases in AWS. Thank you for sticking around if you did.
