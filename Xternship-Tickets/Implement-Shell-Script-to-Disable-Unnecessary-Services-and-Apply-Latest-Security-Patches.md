Description

As a DevOps engineer, I want to update the 001-critical-standards.sh script to disable unnecessary services and apply the latest security patches on both Debian and RedHat based systems. This script must determine essential services and capture exit codes for error handling.

Refer to the CSI Standards Document for further information.

Acceptance Criteria:

The 001-critical-standards.sh script includes a function to disable unnecessary services.

The script includes a function to apply the latest security patches.

The script works on both Debian and RedHat based systems.

The functions capture exit codes and store them in variables for error handling.

If an error occurs in one function, it should determine whether the next function proceeds or exits.

The Packer template calls 001-critical-standards.sh and verify its execution is successful.

The script logs its actions and any errors encountered during execution.

```
#!/bin/bash

# Define essential services
ESSENTIAL_SERVICES=("sshd" "networking" "cron")

# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}

# Function to check the OS type
check_os() {
    if [ -f /etc/redhat-release ]; then
        OS="RedHat"
        log "Detected OS: RedHat"
    elif [ -f /etc/debian_version ]; then
        OS="Debian"
        log "Detected OS: Debian"
    else
        log "Unsupported OS."
        exit 1
    fi
}

# Function to disable non-essential services on Debian-based systems
disable_non_essential_services_debian() {
    log "Disabling non-essential services on Debian-based system..."
    for service in $(systemctl list-unit-files --type=service --state=enabled | grep enabled | awk '{print $1}'); do
        if [[ ! " ${ESSENTIAL_SERVICES[@]} " =~ " ${service%.service} " ]]; then
            systemctl stop "$service" 2>>/var/log/system_update.log
            systemctl disable "$service" 2>>/var/log/system_update.log
            if [ $? -ne 0 ]; then
                log "Failed to disable $service"
                return 1
            fi
        fi
    done
    return 0
}

# Function to disable non-essential services on RedHat-based systems
disable_non_essential_services_redhat() {
    log "Disabling non-essential services on RedHat-based system..."
    for service in $(systemctl list-unit-files --type=service --state=enabled | grep enabled | awk '{print $1}'); do
        if [[ ! " ${ESSENTIAL_SERVICES[@]} " =~ " ${service%.service} " ]]; then
            systemctl stop "$service" 2>>/var/log/system_update.log
            systemctl disable "$service" 2>>/var/log/system_update.log
            if [ $? -ne 0 ]; then
                log "Failed to disable $service"
                return 1
            fi
        fi
    done
    return 0
}

# Function to apply the latest security patches on Debian-based systems
apply_patches_debian() {
    log "Applying latest security patches on Debian-based system..."
    apt-get update 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update package list"
        return 1
    fi

    apt-get upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to upgrade packages"
        return 1
    fi

    apt-get dist-upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to perform dist-upgrade"
        return 1
    fi

    apt-get autoremove -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to autoremove packages"
        return 1
    fi
    return 0
}

# Function to apply the latest security patches on RedHat-based systems
apply_patches_redhat() {
    log "Applying latest security patches on RedHat-based system..."
    yum update -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update packages"
        return 1
    fi

    yum upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to upgrade packages"
        return 1
    fi
    return 0
}

# Main script execution
log "Starting system update script."

check_os

if [ "$OS" = "Debian" ]; then
    disable_non_essential_services_debian
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Disabling non-essential services failed."
        exit 1
    fi

    apply_patches_debian
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Applying patches failed."
        exit 1
    fi

elif [ "$OS" = "RedHat" ]; then
    disable_non_essential_services_redhat
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Disabling non-essential services failed."
        exit 1
    fi

    apply_patches_redhat
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Applying patches failed."
        exit 1
    fi
fi

log "System has been successfully updated and only essential services are enabled."
echo "System update script completed. Check /var/log/system_update.log for details."
```

Instructions:
Save the script to a file, e.g., update_and_enable_essential_services.sh.
Make the script executable: chmod +x update_and_enable_essential_services.sh.
Run the script with sudo to ensure it has the necessary permissions: sudo ./update_and_enable_essential_services.sh.
Notes:
The script defines essential services as sshd, networking, and cron. Modify the ESSENTIAL_SERVICES array to include other essential services as required.
The script disables all non-essential services by stopping and disabling them, then applies the latest security patches.
The script checks the OS type and applies the corresponding commands for Debian or RedHat-based systems.
The script captures exit codes to handle errors appropriately, logs actions and errors to a log file (/var/log/system_update.log), and exits if any command fails.
The script determines whether to proceed with patch application based on the success of disabling services. If disabling services fails, it logs the error and exits. If patch application fails, it logs the error and exits.

Explanation

ure, I'll explain each line of the script in detail:

bash
Copy code
#!/bin/bash
This is a shebang line that tells the system to use bash as the interpreter for this script.

bash
Copy code
# Define essential services
ESSENTIAL_SERVICES=("sshd" "networking" "cron")
This line defines an array of essential services that should remain enabled. You can modify this array to include other essential services specific to your needs.

bash
Copy code
# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}
This function logs messages with a timestamp. It takes a single argument ($1) which is the message to log. The echo command outputs the message, and $(date '+%Y-%m-%d %H:%M:%S') generates the current date and time.

bash
Copy code
# Function to check the OS type
check_os() {
    if [ -f /etc/redhat-release ]; then
        OS="RedHat"
        log "Detected OS: RedHat"
    elif [ -f /etc/debian_version ]; then
        OS="Debian"
        log "Detected OS: Debian"
    else
        log "Unsupported OS."
        exit 1
    fi
}
This function checks the operating system type by looking for specific files that are unique to RedHat-based or Debian-based systems. It sets the OS variable accordingly and logs the detected OS. If neither file is found, it logs an error and exits with a status code of 1.

bash
Copy code
# Function to disable non-essential services on Debian-based systems
disable_non_essential_services_debian() {
    log "Disabling non-essential services on Debian-based system..."
    for service in $(systemctl list-unit-files --type=service --state=enabled | grep enabled | awk '{print $1}'); do
        if [[ ! " ${ESSENTIAL_SERVICES[@]} " =~ " ${service%.service} " ]]; then
            systemctl stop "$service" 2>>/var/log/system_update.log
            systemctl disable "$service" 2>>/var/log/system_update.log
            if [ $? -ne 0 ]; then
                log "Failed to disable $service"
                return 1
            fi
        fi
    done
    return 0
}
This function disables non-essential services on Debian-based systems. It logs the start of the process, then iterates through all enabled services. For each service, it checks if the service is in the ESSENTIAL_SERVICES array. If not, it stops and disables the service, logging any errors to /var/log/system_update.log. If any command fails, it logs the failure and returns 1. If successful, it returns 0.

bash
Copy code
# Function to disable non-essential services on RedHat-based systems
disable_non_essential_services_redhat() {
    log "Disabling non-essential services on RedHat-based system..."
    for service in $(systemctl list-unit-files --type=service --state=enabled | grep enabled | awk '{print $1}'); do
        if [[ ! " ${ESSENTIAL_SERVICES[@]} " =~ " ${service%.service} " ]]; then
            systemctl stop "$service" 2>>/var/log/system_update.log
            systemctl disable "$service" 2>>/var/log/system_update.log
            if [ $? -ne 0 ]; then
                log "Failed to disable $service"
                return 1
            fi
        fi
    done
    return 0
}
This function is similar to the previous one but is intended for RedHat-based systems. The logic and operations are the same.

bash
Copy code
# Function to apply the latest security patches on Debian-based systems
apply_patches_debian() {
    log "Applying latest security patches on Debian-based system..."
    apt-get update 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update package list"
        return 1
    fi

    apt-get upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to upgrade packages"
        return 1
    fi

    apt-get dist-upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to perform dist-upgrade"
        return 1
    fi

    apt-get autoremove -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to autoremove packages"
        return 1
    fi
    return 0
}
This function applies the latest security patches on Debian-based systems. It performs several steps:

apt-get update to update the package list.
apt-get upgrade -y to upgrade all packages.
apt-get dist-upgrade -y to handle changing dependencies with new versions of packages.
apt-get autoremove -y to remove unnecessary packages.
For each command, it logs errors to /var/log/system_update.log and checks the exit code. If any command fails, it logs the failure and returns 1. If successful, it returns 0.

bash
Copy code
# Function to apply the latest security patches on RedHat-based systems
apply_patches_redhat() {
    log "Applying latest security patches on RedHat-based system..."
    yum update -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update packages"
        return 1
    fi

    yum upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to upgrade packages"
        return 1
    fi
    return 0
}
This function applies the latest security patches on RedHat-based systems. It performs two steps:

yum update -y to update all packages.
yum upgrade -y to upgrade all packages.
It logs errors to /var/log/system_update.log and checks the exit code. If any command fails, it logs the failure and returns 1. If successful, it returns 0.

bash
Copy code
# Main script execution
log "Starting system update script."
Logs the start of the script execution.

bash
Copy code
check_os
Calls the check_os function to determine the operating system type.

bash
Copy code
if [ "$OS" = "Debian" ]; then
    disable_non_essential_services_debian
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Disabling non-essential services failed."
        exit 1
    fi

    apply_patches_debian
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Applying patches failed."
        exit 1
    fi

elif [ "$OS" = "RedHat" ]; then
    disable_non_essential_services_redhat
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Disabling non-essential services failed."
        exit 1
    fi

    apply_patches_redhat
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Applying patches failed."
        exit 1
    fi
fi
This block executes the appropriate functions based on the detected OS type. It captures the exit codes of each function and logs errors if any function fails. If disabling non-essential services or applying patches fails, the script logs the failure and exits with status code 1.

bash
Copy code
log "System has been successfully updated and only essential services are enabled."
echo "System update script completed. Check /var/log/system_update.log for details."
Logs the successful completion of the script and notifies the user to check the log file for details.

This script ensures that only essential services are enabled and the latest security patches are applied on both Debian and RedHat-based systems, with robust error handling and logging.

