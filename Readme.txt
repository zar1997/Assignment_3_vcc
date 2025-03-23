Auto-Scaling VM Setup with Resource Monitoring on GCP
This guide provides the steps to set up auto-scaling from an Ubuntu VM to Google Cloud Platform (GCP). The setup includes a resource monitoring script that checks CPU usage on the local VM and triggers auto-scaling to GCP when CPU usage exceeds 75%.

Prerequisites:
Ubuntu VM installed and running.
Google Cloud SDK installed on the VM.
A Google Cloud account with necessary permissions to create instances.
Service account key (in JSON format) for GCP authentication.
Steps to Set Up the System:
1. Update System Packages
Run the following commands to update your system and install necessary packages:

bash
Copy
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates gnupg
2. Authenticate with Google Cloud
Authenticate the Google Cloud SDK using the service account key:

bash
Copy
gcloud auth activate-service-account --key-file=/home/vboxuser/Downloads/key.json
3. List Available GCP Projects
To verify that you are authenticated and see the available GCP projects, run:

bash
Copy
gcloud project list
4. Create the Resource Monitoring Script
Now, create a script that will monitor the CPU usage on your local VM and trigger auto-scaling to GCP when necessary.

Open a new file using nano:

bash
Copy
nano monitor_cpu.sh
Copy and paste the following script into the file:

bash
Copy
# Check CPU usage and scale if > 75%
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/., *\([0-9.]\)%* id.*/\1/" | awk '{print 100 - $1}')
echo "Current CPU usage: $CPU_USAGE%"

if (( $(echo "$CPU_USAGE > 75" | bc -l) )); then
  echo "CPU usage is greater than 75%. Triggering auto-scaling."
  # Trigger auto-scaling (e.g., create a new instance in GCP)
  gcloud compute instances create new-instance --zone=us-central1-a --image-family=debian-9 --image-project=debian-cloud
else
  echo "CPU usage is below 75%. No scaling needed."
fi
Save and close the file by pressing CTRL + X, then Y, and hitting Enter.

5. Make the Script Executable
You need to make the script executable before running it:

bash
Copy
chmod +x monitor_cpu.sh
6. Run the Script
Now, you can run the script to check CPU usage and trigger auto-scaling if necessary:

bash
Copy
./monitor_cpu.sh
Expected Output:
If the CPU usage exceeds 75%, the script will output:

bash
Copy
Current CPU usage: [value]%
CPU usage is greater than 75%. Triggering auto-scaling.
And it will attempt to create a new instance in GCP.

If the CPU usage is below or equal to 75%, it will output:

bash
Copy
Current CPU usage: [value]%
CPU usage is below 75%. No scaling needed.
7. Automate the Script (Optional)
To automate the process of monitoring CPU usage and triggering the script periodically, you can set up a cron job.

Open the crontab editor:

bash
Copy
crontab -e
Add the following line to run the script every minute:

bash
Copy
* * * * * /path/to/your/script/monitor_cpu.sh
Replace /path/to/your/script/monitor_cpu.sh with the actual path to your script.

Conclusion
You have now successfully set up a local Ubuntu VM to monitor its CPU usage and trigger auto-scaling on GCP when necessary. This script can be expanded to monitor additional resources like memory, disk usage, etc., and scale other cloud resources as needed.