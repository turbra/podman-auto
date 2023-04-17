# podman-auto
## How to Auto-starting rootless pods using systemd

### Requirements: 
1. podman
2. ansible
3. running container(s)
4. enable lingering for your user account using:
    ##### Replace `<User>` with your username.
    `loginctl enable-linger <User>`

## Option 1 - Shell Script `add_containers.sh`

Make it executable:
`chmod +x add_containers.sh`

### Run the script:
`./add_containers.sh`
<details>
<summary>Results...</summary>

``` bash
Enter the name of the container you would like to use (or type 'quit' to exit):
hello-friend-container
Created symlink /home/potato/.config/systemd/user/default.target.wants/hello-friend-container.service → /home/potato/.config/systemd/user/hello-friend-container.service.
Successfully created and started the systemd service for container 'hello-friend-container'.
```
<br>
</details>

## Option 2 - Ansible `add_containers.yml`

### Run the playbook
`ansible-playbook add_containers.yml`

Alternatively
`chmod +x add_containers.yml`

`./add_containers.yml`

<details>
<summary>Results...</summary>

``` yaml
Enter the name of the container you would like to use (or type 'quit' to exit):: hello-friend-container

PLAY [Create and start Podman container systemd service] ***************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************
ok: [localhost]

TASK [Get container ID] ************************************************************************************************************************************************************************
changed: [localhost]

TASK [Check if container exists] ***************************************************************************************************************************************************************
skipping: [localhost]

TASK [Check if systemd service already exists] *************************************************************************************************************************************************
changed: [localhost]

TASK [Fail if systemd service already exists] **************************************************************************************************************************************************
skipping: [localhost]

TASK [Create systemd user directory] ***********************************************************************************************************************************************************
ok: [localhost]

TASK [Generate systemd unit file] **************************************************************************************************************************************************************
changed: [localhost]

TASK [Write systemd unit file] *****************************************************************************************************************************************************************
changed: [localhost]

TASK [Enable systemd service] ******************************************************************************************************************************************************************
changed: [localhost]

TASK [Start systemd service] *******************************************************************************************************************************************************************
changed: [localhost]

TASK [Display success message] *****************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Successfully created and started the systemd service for container 'hello-friend-container'."
}

PLAY RECAP *************************************************************************************************************************************************************************************
localhost                  : ok=9    changed=6    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```
<br>
</details>

## Decommission
Ansible `remove_containers.yml`

### Run the playbook
`ansible-playbook remove_containers.yml`

Alternatively
`chmod +x remove_containers.yml`

`./remove_containers.yml`

<details>
<summary>Results...</summary>

``` yaml
Enter the name of the container you would like to decommission (or type 'quit' to exit):: hello-friend-container

PLAY [Remove and stop Podman container systemd service] ****************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************
ok: [localhost]

TASK [Check if systemd unit file exists] *******************************************************************************************************************************************************
ok: [localhost]

TASK [Fail if systemd unit file does not exist] ************************************************************************************************************************************************
skipping: [localhost]

TASK [Stop systemd service] ********************************************************************************************************************************************************************
changed: [localhost]

TASK [Disable systemd service] *****************************************************************************************************************************************************************
changed: [localhost]

TASK [Remove systemd unit file] ****************************************************************************************************************************************************************
changed: [localhost]

TASK [Reload systemd configuration] ************************************************************************************************************************************************************
changed: [localhost]

TASK [Display success message] *****************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Successfully removed and stopped the systemd service for container 'hello-friend-container'."
}

PLAY RECAP *************************************************************************************************************************************************************************************
localhost                  : ok=7    changed=4    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0


```
<br>
</details>

## Note:
If your containers are deployed via CI/CD ensure to change the Service Restart policy to `Restart=unless-stopped` otherwise systemd will start your container as soon as your CI/CD deployment stops it which mayy result with your changes not being applied.
    
`vi ~/.config/systemd/user/hello-friend-container.service`
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
