# Honeypot-
# Week-10-11-Project-Honeypot

Milestone 0: To the Google Cloud!

To complete this assignment, you'll need access to a cloud hosting provider to provision the VMs. Many providers offer time-limited free trial accounts with resource limitations, and you should easily be able to complete the requirements for this assignment within these limitations .
If you're not sure where to start, we recommend Google Cloud Platform's Free Tier, and while we'll provide general guidelines that should work with most cloud providers, the instructions below will also show insets labeled GCP Users with commands and settings specific to Google Cloud Platform. 
Setting up your GCP environment
Navigate to https://console.cloud.google.com/. Log on if necessary.If you have multiple Google Accounts, make sure you are signed in with the account you would like ot use for this project.Click the TRY FOR FREE button. Fill out the information including providing a credit card number. The VMs you create for this project will not use up even 25% of your free credits if you create them as described here and only run them for a week or 2. Google will not autocharge you if you go over your limit.Once you are enrolled you will land on the Welcome page.Create a new project for this assignment.Click the My First Project menu to bring up a list of your projects.Click the NEW PROJECT button.Give the project a name and click CREATE. You will see a notification that the project is being created. This may take a minute.Click the GO TO COMPUTE ENGINE link. You will use this page to create your VMs. It may take a few minutes to prepare your Compute Engine.
To begin, download and install the GCP SDK on your local machine and initialize it such that you can use gcloud from the command line. Be sure to set a default region and zone (these instructions were tested against the us-central1 region and us-central1-c zone).

$ gcloud config set compute/region us-central1
$ gcloud config set compute/zone us-central1-c
Once you've initialized, you should be able to run gcloud config list and see the project, region, and zone configured in the output.

<img width="1552" alt="Screen Shot 2021-11-19 at 6 43 26 PM" src="https://user-images.githubusercontent.com/78555907/142704654-4021775e-1884-4691-9525-c4484ec0f504.png">

Milestone 1: Create MHN Admin VM
First start by creating the MHN Admin VM via your cloud provider.The VM needs to have an internet-facing IP and accessible to you via ssh (or a similar protocol). As specified above, you can use a small or micro-sized VM for this, but the following attributes are required:

Ubuntu 14.04 (trusty)
HTTP traffic allowed (port 80)
TCP ports 3000 and 10000 need to be open to allow incoming (aka 'ingress') traffic That last requirement is generally the only one that may require a specific firewall rule to configure properly, because those ports are non-standard and specific to MHN. Some cloud providers may require you to create the firewall rules separately and then apply them to the VM. Either way, make sure when you create the VM that you can access it via SSH.

<img width="1491" alt="Screen Shot 2021-11-19 at 6 52 58 PM" src="https://user-images.githubusercontent.com/78555907/142705202-a09ed8c7-dd0c-4e8a-87b2-545df36718bc.png">

GCP Users
First, create a firewall rule to allow ingress traffic on TCP ports 3000 and 10000. The following command can be used to do this via the command line:
-$ gcloud beta compute firewall-rules create mhn-allow-admin --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:3000,tcp:10000 --source-ranges=0.0.0.0/0 --target-tags=mhn-admin
This will prompt you to install the beta SDK; after doing so, the command should complete successfully. You can verify the mhn-allow-admin rule was created via the browser by looking at the VPC Network Firewall Ingress Rules. Next, create the VM itself, which we'll call mhn-admin:
-$ gcloud compute instances create "mhn-admin" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-admin","http-server","https-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-admin"
Note the tags value, which controls the applicable firewall rules. The output should show both internal and external IP addresses...make note of the external IP:
<img width="1552" alt="Screen Shot 2021-11-19 at 6 57 57 PM" src="https://user-images.githubusercontent.com/78555907/142705473-06a02fc0-33ab-4b78-aa50-b3b257171316.png">
Finally, establish SSH access to the VM via gcloud compute ssh mhn-admin (which is similar to ssh). You'll be asked to add the fingerprint the first time you connect, and you'll see the Ubuntu welcome message and a command prompt.

Milestone 2: Install the MHN Admin Application
After having established SSH access the MHN Admin VM, the following instructions can be run on the remote VM to install the application. Note: this step may take 30-40 minutes overall. These instructions were adapted from the MHN README. First, update apt and install git:
$ sudo apt-get update
$ sudo apt-get install git -y

Next, clone the MHN code into /opt, change into the clone directory, and run install.sh:
$ cd /opt
$ sudo git clone https://github.com/threatstream/mhn.git
$ cd mhn
$ sudo ./install.sh

You can accept the default values for the rest of the values, either by hitting enter or n for any y/n prompts:
Server base url ["http://#.#.#.#"]:
Honeymap url ["http://#.#.#.#:3000"]:
Mail server address ["localhost"]:
Mail server port [25]:
Use TLS for email?: y/n n
Use SSL for email?: y/n n
Mail server username [""]:
Mail server password [""]:
Mail default sender [""]:
Path for log file ["/var/log/mhn/mhn.log"]:

The script will churn for another 15 minutes or so, and near the end, there will be two more y/n prompts, both of which you can answer n:
Would you like to integrate with Splunk? (y/n) n
Would you like to install ELK? (y/n) n

<img width="1552" alt="Screen Shot 2021-11-19 at 7 02 33 PM" src="https://user-images.githubusercontent.com/78555907/142705693-526a968c-522d-4cfa-b934-6a01ddd8bf7e.png">


Milestone 3: Create a MHN Honeypot VM
First, create a VM for your first honeypot via your cloud provider. As before, this VM also needs to have an internet-facing IP and accessible to you via ssh (or a similar protocol), and as before, you can use a small or micro-sized VM, but it should be running Ubuntu 14.04 (trusty).
Create the VM and establish an SSH connection to it before proceeding to the next step.
GCP Users Run the following commands on your local machine. You can either exit out of the mhn-admin shell or just open a new terminal window. First, create the firewall rule to allow incoming traffic on all ports:
$ gcloud beta compute firewall-rules create mhn-allow-honeypot --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=all --source-ranges=0.0.0.0/0 --target-tags=mhn-honeypot
Now, create the VM for our honeypots, called mhn-honeypot-1 and mhn-honeypot-2: Again, make note of the external IP, then go ahead and establish SSH access to the VM using gcloud compute ssh mhn-honeypot-1 and gcloud compute ssh mhn-honeypot-2. You will again be asked to add the fingerprint and will see the Ubuntu 14.04 welcome message.
gcloud compute instances create "mhn-honeypot-1" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-honeypot","http-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-honeypot-1"

gcloud compute instances create "mhn-honeypot-2" --machine-type "f1-micro" --subnet "default" --maintenance-policy "MIGRATE"  --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring.write","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --tags "mhn-honeypot","http-server" --image "ubuntu-1404-trusty-v20171010" --image-project "ubuntu-os-cloud" --boot-disk-size "10" --boot-disk-type "pd-standard" --boot-disk-device-name "mhn-honeypot-2"

<img width="1552" alt="Screen Shot 2021-11-19 at 7 05 39 PM" src="https://user-images.githubusercontent.com/78555907/142705890-d79f152f-feda-4921-b031-32a0e52bb360.png">

Milestone 4: Install the Honeypot Application

After having established SSH access the new honeypot VM, we need to install the honeypot application into the VM and wire it to connect back to the admin server. Fortunately, MHN makes this fairly straightforward. First, in the MHN admin console in your browser, click on Deploy in the top nav, and you'll be asked to select a script. Choose Ubuntu - Dionaea with HTTP, and you'll see a Deploy Command appear with a full deployment script below it. You can ignore the script, which is just for reference, but make a note of the Deploy Command, which is the one-line command you'll need to execute inside the honeypot VM you connected to in the last step.
So, copy the command from the browser page. It should start with wget and end with a unique token string. Execute this command inside the honeypot VM to install the Dionaea software. It shouldn't take more than a few minutes to complete. When it's done, click back over to the MHN admin console in your browser. From the top nav, choose Sensors >> View sensors and you should see the new honeypot listed.

-wget "http://35.225.22.169/api/script/?text=true&script_id=2" -O deploy.sh && sudo bash deploy.sh http://35.225.22.169 sX9hZta4
-wget "http://104.154.221.36/api/script/?text=true&script_id=6" -O deploy.sh && sudo bash deploy.sh http://104.154.221.36 wjT0luE0

<img width="1012" alt="Screen Shot 2021-11-19 at 7 30 12 PM" src="https://user-images.githubusercontent.com/78555907/142707269-b4bb29b4-ce3f-420d-bf34-d4b75ea4384c.png">

Note : I was not able to execute this command inside the honeypot VM to install the Dionaea software.I attached the screenshot of the error.







