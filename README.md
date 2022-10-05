# Introduction

When migrating from one kubernetes cluster to another, I needed to sync the files that were stored in a volume from one cluster to the other.

To do this, I used rsync. This chart is simple, it just deployes a pod that has rsync and ssh installed in it. It has some variables for specifying the
ssh key, the target IP addresses, and some other information.

# Usage

To use this, assert that there are two Kubernetes Clusters: Cluster A and Cluster B. Cluster A has the data in a volume that needs to be
copied to Cluster B.

1. Alter the deployment's volumes so that it has read-only access to the volumes in Cluster A that you want to sync to Cluster B.

1. Create a secret.yaml file that has the ssh key that will be used when creating a tunnel from Cluster A to Cluster B.

1. Deploy the chart to Cluster A. It runs the sleep command infinitely, so it will be running. I used this:

   `helm upgrade --install -f ./rsync-files/values.yaml -f ./rsyn-files/secrets.yaml rsync-files ./rsync-files --debug`

1. Use `kubectl exec -it <pod name> -- bash` to enter the cluster in order to run the rsync command.

1. Create a machine in Cluster B or one that has access to Cluster B's volumes and expose it to the world using an IP address or dns name.

   _This doesn't necessarily need to be a pod. I created a temporary vm that mounted the filestore that was being used in the cluster. It was simpler and easier than creating the appropriate ingress resources/managed certificates/frontend and backend configs/static public ip addresses and dns entries._

1. Update the deployment's `values.yaml` with the target IP address or Domain name for the target machine.

1. Update the deployment's `secrets.example.yaml` with the ssh key of the user that is present on the target machine.

1. Verify that you can ssh from the pod to the other machine: `ssh -u $SSH_USER@$REMOTE_RSYNC_IP`

1. Launch tmux to run the rsync: `tmux new -s rsync_files`

1. Run the rsync: `rsync -ravzpP --bwlimit=$BANDWIDTH_LIMIT_KBS --delete -e "ssh -o StrictHostKeyChecking=no" --rsync-path="sudo rsync" $SOURCE_DIRECTORY $SSh_USER@$REMOTE_RSYNC_IP:$DESTINATION_DIRECTORY`
