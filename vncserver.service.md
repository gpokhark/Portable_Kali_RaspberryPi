# `vncser.service`
- This file is provided by chatGPT and not tested by me
    ```bash
    [Unit]
    Description=VNC remote desktop server
    After=sshd.service
    
    [Service]
    Type=simple
    ExecStart=/usr/bin/vncserver -localhost no -geometry 1280x800
    ExecStop=/usr/bin/vncserver -kill :%i
    User=kali
    Restart=on-failure
    RestartSec=3
    
    [Install]
    WantedBy=multi-user.target
    ```

- Changes made:
    - Changed `Type=dbus` to `Type=simple`: The `simple` type is appropriate for services that do not fork into the background. Since VNC server runs in the foreground, `simple` is more suitable.
    - Added `-localhost no -geometry 1280x800` to the ExecStart line: This ensures that the VNC server is accessible from remote machines (`-localhost no`) and sets the default screen resolution to 1280x800. You can adjust the geometry based on your preference.
    - Removed the duplicate `Type=forking` line: It was unnecessary since `Type=simple` is used.
    - Added `Restart=on-failure` and `RestartSec=3`: This ensures that the service will be restarted in case of failure, with a 3-second delay between restarts.

- Following works -

    ```bash
    [Unit]
    Description=VNC remote desktop server
    After=sshd.service
    
    [Service]
    Type=dbus
    ExecStart=/usr/bin/vncserver
    ExecStop=/usr/bin/vncserver -kill :%i
    User=kali
    Restart=on-failure
    RestartSec=3
    Type=forking
    
    [Install]
    WantedBy=multi-user.target
    ```