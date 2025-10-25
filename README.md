# Dynamic DNS Rotator

This script automates the process of rotating through a predefined list of public DNS servers on your Debian/Ubuntu-based system. It continuously updates your `/etc/resolv.conf` file with new DNS configurations every two minutes, cycling through various DNS providers.

## ⚠️ WARNING: Use with Caution ⚠️

This script **continuously modifies your system's DNS configuration**.
*   **Understand the implications:** Changing DNS frequently can impact network performance, caching, and potentially lead to temporary connectivity issues if a chosen DNS server is slow or unresponsive.
*   **Persistent Changes:** While this script modifies `/etc/resolv.conf`, on many modern systems (especially those using `systemd-resolved`, `NetworkManager`, or `netplan`), `/etc/resolv.conf` is often a symlink or is dynamically managed. Direct edits to this file might be overwritten by these services. You may need to disable or reconfigure these services if you want this script's changes to be truly persistent or to avoid conflicts.
*   **Backup:** It's always a good idea to back up your original `resolv.conf` file before running the script:
    ```bash
    sudo cp /etc/resolv.conf /etc/resolv.conf.bak
    ```
*   **Stopping the script:** Remember that this script runs in an infinite loop. You will need to manually stop it (e.g., by pressing `Ctrl+C` in the terminal or killing the process) when you no longer want it to run.

## Features

*   **Automatic DNS Rotation:** Cycles through a comprehensive list of public DNS providers.
*   **Timed Updates:** Changes DNS every 2 minutes.
*   **Colorful Output:** Uses ANSI escape codes for improved readability in the terminal.
*   **Sudo Integration:** Prompts for `sudo` to modify `/etc/resolv.conf`.

## Prerequisites

*   A Debian or Ubuntu-based operating system.
*   `sudo` privileges.
*   Basic understanding of how DNS works and how your system manages `/etc/resolv.conf`.

## How to Use

1.  **Save the Script:**
    Save the provided script content into a file, for example, `dns_rotator.sh`.

    ```bash
    #!/bin/bash

    # Colors for output
    RED='\033[0;31m'
    GREEN='\033[0;32m'
    YELLOW='\033[0;33m'
    BLUE='\033[0;34m'
    CYAN='\033[0;36m'
    RESET='\033[0m'

    # List of DNS servers (Provider: Primary DNS : Secondary DNS)
    # Note: Some providers might only have a primary DNS listed.
    DNS_SERVERS=(
        "Google Public DNS:8.8.8.8:8.8.4.4"
        "Cloudflare:1.1.1.1:1.0.0.1"
        "OpenDNS:208.67.222.222:208.67.220.220"
        "Level 3:209.244.0.3:209.244.0.4"
        "Verisign:64.6.64.6:64.6.65.6"
        "Quad9:9.9.9.9:149.112.112.112"
        "Comodo Secure DNS:8.26.56.26:8.20.247.20"
        "DNS.WATCH:84.200.69.80:84.200.70.40"
        "Norton ConnectSafe:199.85.126.10:199.85.127.10"
        "GreenTeamDNS:81.218.119.11:209.88.198.133"
        "SafeDNS:195.46.39.39:195.46.39.40"
        "OpenNIC:185.121.177.177:169.239.202.202"
        "SmartViper:208.76.50.50:208.76.51.51"
        "Freenom World:80.80.80.80:80.80.81.81"
        "Dyn:216.146.35.35:216.146.36.36"
        "FreeDNS:37.235.1.174:37.235.1.177"
        "Alternate DNS:198.101.242.72:23.253.163.53"
        "Yandex.DNS:77.88.8.8:77.88.8.1"
        "UncensoredDNS:91.239.100.100:89.233.43.71"
        "Hurricane Electric:74.82.42.42:" # No secondary DNS provided for Hurricane Electric in the original script
        "Neustar:156.154.70.1:156.154.71.1"
        "CNNIC SDNS:1.2.4.8:210.2.4.8"
        "AliDNS:223.5.5.5:223.6.6.6"
        "Baidu Public DNS:180.76.76.76:" # No secondary DNS provided for Baidu in the original script
        "DNSPod Public DNS+:119.29.29.29:119.28.28.28"
        "114DNS:114.114.114.114:114.114.115.115"
        "OneDNS:117.50.11.11:117.50.22.22"
        "DNSpai:101.226.4.6:218.30.118.6"
    )

    # Path to the resolv.conf file
    RESOLV_CONF="/etc/resolv.conf"

    # Function to update DNS
    update_dns() {
        local primary=$1
        local secondary=$2

        echo -e "${CYAN}Writing DNS servers to /etc/resolv.conf...${RESET}"
        echo -e "${GREEN}Primary DNS: ${primary}${RESET}"
        if [[ -n $secondary ]]; then
            echo -e "${GREEN}Secondary DNS: ${secondary}${RESET}"
        else
            echo -e "${YELLOW}Secondary DNS: Not provided (or intentionally left blank)${RESET}"
        fi

        # Write to /etc/resolv.conf
        # Using a here-document with 'sudo bash -c' to write to the file as root
        sudo bash -c "cat > $RESOLV_CONF" <<EOL
    nameserver $primary
    $(if [[ -n "$secondary" ]]; then echo "nameserver $secondary"; fi)
    EOL

        echo -e "${BLUE}DNS servers updated successfully!${RESET}"
    }

    # Main loop to rotate through DNS servers every 2 minutes
    while true; do
        for dns_entry in "${DNS_SERVERS[@]}"; do
            IFS=":" read -r provider primary secondary <<< "$dns_entry"

            # Display details about the current DNS being applied
            echo -e "${YELLOW}-----------------------------------------------${RESET}"
            echo -e "${BLUE}Switching DNS to provider: ${CYAN}${provider}${RESET}"
            echo -e "${YELLOW}-----------------------------------------------${RESET}"

            # Update DNS servers
            # The update_dns function now handles cases where secondary is empty
            update_dns "$primary" "$secondary"

            # Display a detailed message about the next step
            echo -e "${CYAN}Waiting for 2 minutes before the next update...${RESET}"
            sleep 120 # Wait 2 minutes
        done
    done
    ```

2.  **Make it Executable:**
    ```bash
    chmod +x dns_rotator.sh
    ```

3.  **Run the Script:**
    ```bash
    sudo ./dns_rotator.sh
    ```
    The script will start rotating DNS servers immediately and continue indefinitely until stopped.

## Stopping the Script

To stop the script, simply press `Ctrl+C` in the terminal where it is running.

If you ran it in the background or detached it from the terminal, you might need to find its process ID (PID) and kill it:

1.  Find the process:
    ```bash
    ps aux | grep dns_rotator.sh
    ```
    Look for a line containing `bash ./dns_rotator.sh` and note the PID (the second column).

2.  Kill the process:
    ```bash
    sudo kill <PID>
    ```
    (Replace `<PID>` with the actual process ID you found).

## After Stopping

After stopping the script, your `/etc/resolv.conf` will remain set to the last DNS servers applied by the script. If you want to revert to your system's default or previous DNS configuration:

1.  **Restore from backup** (if you made one):
    ```bash
    sudo mv /etc/resolv.conf.bak /etc/resolv.conf
    ```
2.  **Restart your networking service** (this might depend on your system):
    *   For `systemd-resolved` based systems:
        ```bash
        sudo systemctl restart systemd-resolved
        ```
    *   For `NetworkManager` based systems:
        ```bash
        sudo systemctl restart NetworkManager
        ```
    *   Or simply reboot your system.

## Customizing DNS Servers

You can easily customize the list of DNS servers by editing the `DNS_SERVERS` array in the script. Each entry should follow the format: `"Provider Name:Primary_IP:Secondary_IP"`. If a secondary IP is not available, you can leave it blank (e.g., `"Provider Name:Primary_IP:"`).

## Potential Conflicts with Network Management Services

Modern Linux distributions often use services like `systemd-resolved`, `NetworkManager`, or `netplan` to manage network configurations, including DNS. These services might overwrite changes made directly to `/etc/resolv.conf`.

*   If you find that your DNS settings are being reverted, you might need to:
    *   **Disable the service** (e.g., `sudo systemctl stop systemd-resolved` and `sudo systemctl disable systemd-resolved`). **This is generally NOT recommended** as it can break other network functionalities.
    *   **Reconfigure the service** to use `stub-resolv.conf` or a custom `resolv.conf` path, or to specifically *not* manage `/etc/resolv.conf`. This is a more advanced task and varies greatly by system configuration.

For most users, simply running the script and being aware of its temporary nature or manually restoring `resolv.conf` afterward will suffice.

## License

This script is provided under the MIT License.
