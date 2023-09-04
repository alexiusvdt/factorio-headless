[6yr guide](https://www.reddit.com/r/factorio/comments/6qo2ge/guide_setting_up_an_ubuntu_headless_server/?st=JQM7K23U&sh=edd52687)
[Digital O](https://docs.digitalocean.com/developer-center/setting-up-a-factorio-multiplayer-server-on-digitalocean/)
[admins for server](https://forums.factorio.com/viewtopic.php?t=65548)

# Create Droplet
  * add SSH key(s)
  * ```apt-get update
        apt-get upgrade -y
    ```

# allow SSH
  ```
  ufw allow openssh
  ufw allow 34197/update
  ufw enable
  ```

# add fail2ban
  * `apt-get install -y fail2ban`

# set server timezone (optional, for logging purposes)  
  * `dpkg-reconfigure tzdata`

# add headless pkg
`wget -O factorio_headless.tar.gz https://www.factorio.com/get-download/1.1.87/headless/linux64` <- as of 9/2/23
use ls to verify correct dir, should show bin, dev, factorio_headless, lib, etc
`cd opt` 
`sudo tar -xf /factorio_headless.tar.gz`
generate a map `./bin/x64/factorio --create ./saves/digitalocean.zip`

# start headless server
`./bin/x64/factorio --start-server digitalocean.zip`

OR 

# create a service
(creating a service has a dedicated account (NOT ROOT) running the svc & has ownership of the files. this is much more secure)
  * navigate to data dir in factorio to look at `server-settings.example.json`
  * run it and remove the .example extension`cp server-settings.example.json server-settings.json`
  * add a new user and give them ownership of headless files:
    ```
    useradd factorio
    chown -R factorio:factorio /opt/factorio
    ```
  * create the file: `sudo nano /etc/systemd/system/factorio.service`
  * pop this in: note: change --start-server with desired savefile name
    ```
    [Unit]
    Description=Factorio Headless Server
    [Service]
    Type=simple
    User=factorio
    ExecStart=/opt/factorio/bin/x64/factorio --start-server /opt/factorio/saves/YOUR_SAVEFILE_NAME.zip --server-settings /opt/factorio/data/server-settings.json
    ```
  * save & exit preserving the filename
  * to start this service, reboot the svc daemon and run it again:
    ```
    systemctl daemon-reload
    systemctl start factorio
    ```
  * verify status with `systemctl status factorio`
  * shut it down with `sudo systemctl stop factorio`

# modifying jsons
  * cd to `/opt/factorio/data`
  * open `nano server-settings.json` for various player settings, LAN visibility, etc
  <!-- * lots of options here, make sure you add yourself as admin by either modifying/creating: -->
  <!-- ```
  {
  ...
  "_comment_max_players": "Maximum number of players allowed, admins can join even a full server. 0 means unlimited.",
  {
  },
  "_comment_allow_commands": "possible values are, true, false and admins-only",
  "allow_commands": "admins-only",
  "only_admins_can_pause_the_game": true,
  "_comment_admins": "List of case insensitive usernames, that will be promoted immediately",
  "admins": [admin1, admin2, etc]
  }
  ``` -->
  this supposedly works, but I can't get it to. to actually add admin:
  * add `server-adminlist.json` to `/opt/factorio/` as an arr:
  ```
  [
    "name1",
    "name2"
  ]
  ```

# addming mods
  * inside `/opt/factorio/mods` (may already exist)
  * note: mods do not auto update. There are py scripts that will run chron jobs to check like [this](https://github.com/astevens/factorio-mod-updater)
  * add all mod zips plus a `mod-list.json`
  * [this](https://github.com/mickael9/fac) tool helps
  * or just remote file upload using filezilla