### Logging Function
```sh
# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a /var/log/system_update.log
}
```
- **Purpose**: This function logs messages with timestamps.
- **Explanation**:
  - `log() { ... }`: Defines a shell function named `log`.
  - `date '+%Y-%m-%d %H:%M:%S'`: Generates the current date and time in the format `YYYY-MM-DD HH:MM:SS`.
  - `echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"`: Prints the current timestamp followed by the message passed as an argument to the function (`$1`).
  - `| tee -a /var/log/system_update.log`: Appends the log message to the file `/var/log/system_update.log` and also prints it to the terminal.

### OS Detection and Defining Non-Essential Services
```sh
# Function to check the OS type and define non-essential services
check_os() {
    if [ -f /etc/redhat-release ]; then
        OS="RedHat"
        log "Detected OS: RedHat"
        NON_ESSENTIAL_SERVICES=("rhsmcertd" "snapd" "polkit" "acpid" "snap.amazon-ssm-agent.amazon-ssm-agent")
    elif [ -f /etc/debian_version ]; then
        OS="Debian"
        log "Detected OS: Debian"
        NON_ESSENTIAL_SERVICES=("snapd" "polkit" "acpid")
    else
        log "Unsupported OS."
        exit 1
    fi
}
```
- **Purpose**: This function detects the operating system type and defines a list of non-essential services for that OS.
- **Explanation**:
  - `if [ -f /etc/redhat-release ]; then`: Checks if the file `/etc/redhat-release` exists, which indicates a RedHat-based system.
    - `OS="RedHat"`: Sets the variable `OS` to "RedHat".
    - `log "Detected OS: RedHat"`: Logs the detected OS.
    - `NON_ESSENTIAL_SERVICES=(...)`: Defines an array of non-essential services for RedHat.
  - `elif [ -f /etc/debian_version ]; then`: Checks if the file `/etc/debian_version` exists, indicating a Debian-based system.
    - `OS="Debian"`: Sets the variable `OS` to "Debian".
    - `log "Detected OS: Debian"`: Logs the detected OS.
    - `NON_ESSENTIAL_SERVICES=(...)`: Defines an array of non-essential services for Debian.
  - `else`: If neither file is found, the script assumes the OS is unsupported.
    - `log "Unsupported OS."`: Logs that the OS is unsupported.
    - `exit 1`: Exits the script with an error code 1.

### Disabling Non-Essential Services for Debian
```sh
# Function to disable non-essential services on Debian-based systems
disable_non_essential_services_debian() {
    log "Disabling non-essential services on Debian-based system..."
    for service in "${NON_ESSENTIAL_SERVICES[@]}"; do
        systemctl stop "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to stop $service"
            continue
        fi
        systemctl disable "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to disable $service"
            continue
        fi
    done
    return 0
}
```
- **Purpose**: This function stops and disables non-essential services on Debian-based systems.
- **Explanation**:
  - `log "Disabling non-essential services on Debian-based system..."`: Logs the action.
  - `for service in "${NON_ESSENTIAL_SERVICES[@]}"; do`: Iterates over each service in the `NON_ESSENTIAL_SERVICES` array.
    - `systemctl stop "$service" 2>>/var/log/system_update.log`: Attempts to stop the service, appending any error messages to the log file.
    - `if [ $? -ne 0 ]; then`: Checks if the `systemctl stop` command failed.
      - `log "Failed to stop $service"`: Logs the failure.
      - `continue`: Skips to the next service in the loop.
    - `systemctl disable "$service" 2>>/var/log/system_update.log`: Attempts to disable the service, appending any error messages to the log file.
    - `if [ $? -ne 0 ]; then`: Checks if the `systemctl disable` command failed.
      - `log "Failed to disable $service"`: Logs the failure.
      - `continue`: Skips to the next service in the loop.
  - `done`: Ends the loop.
  - `return 0`: Returns success.

### Disabling Non-Essential Services for RedHat
```sh
# Function to disable non-essential services on RedHat-based systems
disable_non_essential_services_redhat() {
    log "Disabling non-essential services on RedHat-based system..."
    for service in "${NON_ESSENTIAL_SERVICES[@]}"; do
        systemctl stop "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to stop $service"
            continue
        fi
        systemctl disable "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to disable $service"
            continue
        fi
    done
    return 0
}
```
- **Purpose**: This function stops and disables non-essential services on RedHat-based systems.
- **Explanation**: Identical to the Debian function but operates on RedHat-based systems.

### Cleaning and Updating Package Repository on Debian
```sh
# Function to clean and update the package repository on Debian-based systems
clean_update_repo_debian() {
    log "Cleaning and updating package repository on Debian-based system..."
    rm -rf /var/lib/apt/lists/* 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to clean package lists"
        return 1
    fi

    apt-get clean 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to clean apt cache"
        return 1
    fi

    apt-get update -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update package lists"
        return 1
    fi

    return 0
}
```
- **Purpose**: This function cleans and updates the package repository on Debian-based systems.
- **Explanation**:
  - `log "Cleaning and updating package repository on Debian-based system..."`: Logs the action.
  - `rm -rf /var/lib/apt/lists/* 2>>/var/log/system_update.log`: Removes the package lists directory, logging any errors.
  - `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
    - `log "Failed to clean package lists"`: Logs the failure.
    - `return 1`: Returns an error code.
  - `apt-get clean 2>>/var/log/system_update.log`: Cleans the apt cache, logging any errors.
  - `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
    - `log "Failed to clean apt cache"`: Logs the failure.
    - `return 1`: Returns an error code.
  - `apt-get update -y 2>>/var/log/system_update.log`: Updates the package lists, logging any errors.
  - `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
    - `log "Failed to update package lists"`: Logs the failure.
    - `return 1`: Returns an error code.
  - `return 0`: Returns success.

### Applying Security Patches on Debian
```sh
# Function to apply the latest security patches on Debian-based systems
apply_patches_debian() {
    log "Applying latest security patches on Debian-based system..."
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
```
- **Purpose**: This function applies the latest security patches on Debian-based systems.
- **Explanation**:
  - `log "Applying latest security patches on Debian-based system..."`: Logs the action.
  - `apt-get upgrade -y 2>>/var/log/system_update.log`: Upgrades all packages, logging any errors.
  - `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
    - `log "Failed to


Sure, here's a detailed explanation of each line of the script:

### Logging Function
```sh
# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a /var/log/system_update.log
}
```
- `log() { ... }`: Defines a function named `log`.
- `echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"`: Prints the current date and time followed by the message passed as an argument to the function.
- `| tee -a /var/log/system_update.log`: Appends the output to `/var/log/system_update.log` and also prints it to the terminal.

### OS Detection and Defining Non-Essential Services
```sh
# Function to check the OS type and define non-essential services
check_os() {
    if [ -f /etc/redhat-release ]; then
        OS="RedHat"
        log "Detected OS: RedHat"
        NON_ESSENTIAL_SERVICES=("rhsmcertd" "snapd" "polkit" "acpid" "snap.amazon-ssm-agent.amazon-ssm-agent")
    elif [ -f /etc/debian_version ]; then
        OS="Debian"
        log "Detected OS: Debian"
        NON_ESSENTIAL_SERVICES=("snapd" "polkit" "acpid")
    else
        log "Unsupported OS."
        exit 1
    fi
}
```
- `check_os() { ... }`: Defines a function named `check_os`.
- `if [ -f /etc/redhat-release ]; then`: Checks if the file `/etc/redhat-release` exists, which indicates a RedHat-based system.
- `OS="RedHat"`: Sets the `OS` variable to "RedHat".
- `log "Detected OS: RedHat"`: Logs that the OS is RedHat.
- `NON_ESSENTIAL_SERVICES=("rhsmcertd" "snapd" "polkit" "acpid" "snap.amazon-ssm-agent.amazon-ssm-agent")`: Defines an array of non-essential services for RedHat.
- `elif [ -f /etc/debian_version ]; then`: Checks if the file `/etc/debian_version` exists, indicating a Debian-based system.
- `OS="Debian"`: Sets the `OS` variable to "Debian".
- `log "Detected OS: Debian"`: Logs that the OS is Debian.
- `NON_ESSENTIAL_SERVICES=("snapd" "polkit" "acpid")`: Defines an array of non-essential services for Debian.
- `else`: If neither file is found, the OS is unsupported.
- `log "Unsupported OS."`: Logs that the OS is unsupported.
- `exit 1`: Exits the script with an error code.

### Disabling Non-Essential Services for Debian
```sh
# Function to disable non-essential services on Debian-based systems
disable_non_essential_services_debian() {
    log "Disabling non-essential services on Debian-based system..."
    for service in "${NON_ESSENTIAL_SERVICES[@]}"; do
        systemctl stop "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to stop $service"
            continue
        fi
        systemctl disable "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to disable $service"
            continue
        fi
    done
    return 0
}
```
- `disable_non_essential_services_debian() { ... }`: Defines a function named `disable_non_essential_services_debian`.
- `log "Disabling non-essential services on Debian-based system..."`: Logs the action of disabling non-essential services.
- `for service in "${NON_ESSENTIAL_SERVICES[@]}"; do`: Iterates over each service in the `NON_ESSENTIAL_SERVICES` array.
- `systemctl stop "$service" 2>>/var/log/system_update.log`: Attempts to stop the service and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to stop $service"`: Logs the failure to stop the service.
- `continue`: Skips to the next iteration of the loop.
- `systemctl disable "$service" 2>>/var/log/system_update.log`: Attempts to disable the service and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to disable $service"`: Logs the failure to disable the service.
- `continue`: Skips to the next iteration of the loop.
- `done`: Ends the loop.
- `return 0`: Returns success.

### Disabling Non-Essential Services for RedHat
```sh
# Function to disable non-essential services on RedHat-based systems
disable_non_essential_services_redhat() {
    log "Disabling non-essential services on RedHat-based system..."
    for service in "${NON_ESSENTIAL_SERVICES[@]}"; do
        systemctl stop "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to stop $service"
            continue
        fi
        systemctl disable "$service" 2>>/var/log/system_update.log
        if [ $? -ne 0 ]; then
            log "Failed to disable $service"
            continue
        fi
    done
    return 0
}
```
- Identical to the Debian function, but for RedHat-based systems.

### Cleaning and Updating Package Repository on Debian
```sh
# Function to clean and update the package repository on Debian-based systems
clean_update_repo_debian() {
    log "Cleaning and updating package repository on Debian-based system..."
    rm -rf /var/lib/apt/lists/* 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to clean package lists"
        return 1
    fi

    apt-get clean 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to clean apt cache"
        return 1
    fi

    apt-get update -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update package lists"
        return 1
    fi

    return 0
}
```
- `clean_update_repo_debian() { ... }`: Defines a function named `clean_update_repo_debian`.
- `log "Cleaning and updating package repository on Debian-based system..."`: Logs the action of cleaning and updating the package repository.
- `rm -rf /var/lib/apt/lists/* 2>>/var/log/system_update.log`: Removes the package lists and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to clean package lists"`: Logs the failure to clean package lists.
- `return 1`: Returns an error code.
- `apt-get clean 2>>/var/log/system_update.log`: Cleans the apt cache and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to clean apt cache"`: Logs the failure to clean apt cache.
- `return 1`: Returns an error code.
- `apt-get update -y 2>>/var/log/system_update.log`: Updates the package lists and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to update package lists"`: Logs the failure to update package lists.
- `return 1`: Returns an error code.
- `return 0`: Returns success.

### Applying Security Patches on Debian
```sh
# Function to apply the latest security patches on Debian-based systems
apply_patches_debian() {
    log "Applying latest security patches on Debian-based system..."
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
```
- `apply_patches_debian() { ... }`: Defines a function named `apply_patches_debian`.
- `log "Applying latest security patches on Debian-based system..."`: Logs the action of applying security patches.
- `apt-get upgrade -y 2>>/var/log/system_update.log`: Upgrades packages and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to upgrade packages"`: Logs the failure to upgrade packages.
- `return 1`: Returns an error code.
- `apt-get dist-upgrade -y 2>>/var/log/system_update.log`: Performs a distribution upgrade and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to perform dist-upgrade"`: Logs the

 failure to perform dist-upgrade.
- `return 1`: Returns an error code.
- `apt-get autoremove -y 2>>/var/log/system_update.log`: Removes unnecessary packages and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to autoremove packages"`: Logs the failure to autoremove packages.
- `return 1`: Returns an error code.
- `return 0`: Returns success.

### Applying Security Patches on RedHat
```sh
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
```
- `apply_patches_redhat() { ... }`: Defines a function named `apply_patches_redhat`.
- `log "Applying latest security patches on RedHat-based system..."`: Logs the action of applying security patches.
- `yum update -y 2>>/var/log/system_update.log`: Updates packages and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to update packages"`: Logs the failure to update packages.
- `return 1`: Returns an error code.
- `yum upgrade -y 2>>/var/log/system_update.log`: Upgrades packages and logs any errors.
- `if [ $? -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to upgrade packages"`: Logs the failure to upgrade packages.
- `return 1`: Returns an error code.
- `return 0`: Returns success.

### Main Script Execution
```sh
# Main script execution
log "Starting system update script."

check_os

if [ "$OS" = "Debian" ]; then
    clean_update_repo_debian
    CLEAN_UPDATE_EXIT_CODE=$?
    if [ $CLEAN_UPDATE_EXIT_CODE -ne 0 ]; then
        log "Failed to clean and update package repository. Exiting..."
        exit 1
    fi

    disable_non_essential_services_debian
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Failed to disable unnecessary services. Exiting..."
        exit 1
    fi

    apply_patches_debian
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Failed to apply security patches. Exiting..."
        exit 1
    fi

elif [ "$OS" = "RedHat" ]; then
    disable_non_essential_services_redhat
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Failed to disable unnecessary services. Exiting..."
        exit 1
    fi

    apply_patches_redhat
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Failed to apply security patches. Exiting..."
        exit 1
    fi
fi

log "System has been successfully updated and only essential services are enabled."
echo "System update script successfully completed. Check /var/log/system_update.log for details."
```
- `log "Starting system update script."`: Logs the start of the script.
- `check_os`: Calls the `check_os` function to detect the OS and define non-essential services.
- `if [ "$OS" = "Debian" ]; then`: Checks if the detected OS is Debian.
- `clean_update_repo_debian`: Calls the function to clean and update the package repository for Debian.
- `CLEAN_UPDATE_EXIT_CODE=$?`: Captures the exit code of the previous command.
- `if [ $CLEAN_UPDATE_EXIT_CODE -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to clean and update package repository. Exiting..."`: Logs the failure and exits the script.
- `exit 1`: Exits the script with an error code.
- `disable_non_essential_services_debian`: Calls the function to disable non-essential services for Debian.
- `DISABLE_EXIT_CODE=$?`: Captures the exit code of the previous command.
- `if [ $DISABLE_EXIT_CODE -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to disable unnecessary services. Exiting..."`: Logs the failure and exits the script.
- `exit 1`: Exits the script with an error code.
- `apply_patches_debian`: Calls the function to apply security patches for Debian.
- `PATCH_EXIT_CODE=$?`: Captures the exit code of the previous command.
- `if [ $PATCH_EXIT_CODE -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to apply security patches. Exiting..."`: Logs the failure and exits the script.
- `exit 1`: Exits the script with an error code.
- `elif [ "$OS" = "RedHat" ]; then`: Checks if the detected OS is RedHat.
- `disable_non_essential_services_redhat`: Calls the function to disable non-essential services for RedHat.
- `DISABLE_EXIT_CODE=$?`: Captures the exit code of the previous command.
- `if [ $DISABLE_EXIT_CODE -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to disable unnecessary services. Exiting..."`: Logs the failure and exits the script.
- `exit 1`: Exits the script with an error code.
- `apply_patches_redhat`: Calls the function to apply security patches for RedHat.
- `PATCH_EXIT_CODE=$?`: Captures the exit code of the previous command.
- `if [ $PATCH_EXIT_CODE -ne 0 ]; then`: Checks if the previous command failed.
- `log "Failed to apply security patches. Exiting..."`: Logs the failure and exits the script.
- `exit 1`: Exits the script with an error code.
- `log "System has been successfully updated and only essential services are enabled."`: Logs the successful completion of the script.
- `echo "System update script successfully completed. Check /var/log/system_update.log for details."`: Prints a completion message to the terminal.
