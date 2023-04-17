# podman-auto
How to Auto-starting rootless pods using systemd

### IMPORTANT: Before running this script, make sure to enable lingering for your user account using:
#### Replace <User> with your username.
`loginctl enable-linger <User>`


### Create a script named add_containers.sh and open it with your preferred text editor, like vi. 
#### This script will be responsible for generating systemd unit files and enabling them for your current user.

``` bash
#!/bin/bash

function check_systemd_service_exists() {
    local container_name="$1"
    if systemctl --user list-unit-files --no-legend --type=service | grep -q "^$container_name\.service"; then
        return 0
    else
        return 1
    fi
}

function get_container_id() {
    local container_name="$1"
    podman ps -a --filter "name=$container_name" --format "{{.ID}}" | head -n 1
}

while true; do
    echo "Enter the name of the container you would like to use (or type 'quit' to exit):"
    read container_name

    if [[ $container_name == "quit" ]]; then
        echo "Exiting..."
        exit 0
    elif check_systemd_service_exists "$container_name"; then
        echo "Systemd service for container '$container_name' already exists. Please try again."
    else
        break
    fi
done

# Generate systemd unit file
container_id=$(get_container_id "$container_name")

if [ -z "$container_id" ]; then
    echo "No container found with the name '$container_name'. Please try again."
    exit 1
fi

user_systemd_dir="$HOME/.config/systemd/user"
mkdir -p "$user_systemd_dir"
podman generate systemd --new --name "$container_id" > "$user_systemd_dir/$container_name.service"

if [ $? -ne 0 ]; then
    echo "Failed to generate systemd unit file for container '$container_name'."
    exit 1
fi

# Enable and start the service
systemctl --user enable "$container_name"
systemctl --user start "$container_name"

echo "Successfully created and started the systemd service for container '$container_name'."
```

### Save the script and make it executable:
`chmod +x start_containers.sh`

### Run the script:
`./add_containers.sh`
```
Enter the name of the container you would like to use (or type 'quit' to exit):
hello-friend-container
Created symlink /home/potato/.config/systemd/user/default.target.wants/hello-friend-container.service → /home/potato/.config/systemd/user/hello-friend-container.service.
Successfully created and started the systemd service for container 'hello-friend-container'.
```

### Note:
If your containers are deployed via CI/CD ensure to change the Service Restart policy to `Restart=unless-stopped` otherwise systemd will start your container as soon as your CI/CD deployment stops it which mayy result with your changes not being applied.
    
`vi .config/systemd/user/hello-friend-container.service`
```
...
[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=unless-stopped
TimeoutStopSec=70
...
```
### After updating your Systemd unit file, ensure to run the command below to ensure your changes take effect: 
`systemctl daemon-reload`
