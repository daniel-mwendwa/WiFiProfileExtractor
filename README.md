# WiFi Profiles Retriever

This script retrieves the saved WiFi profiles on a Windows machine and their corresponding passwords (if available). It uses the `netsh` command to interact with the WLAN profiles and extracts information using Python's `subprocess` and `re` modules.

## Features
- Lists all saved WiFi profiles on the system.
- Extracts the SSID (WiFi name) and password for profiles with a security key.
- Skips profiles without a password.

## Prerequisites
- This script works on Windows operating systems.
- Python 3.x must be installed.

## Dependencies
- Python's built-in libraries: `subprocess`, `re`.

## How It Works
1. The script uses `netsh wlan show profile` to list all saved WiFi profiles.
2. It filters profiles with security keys using regex.
3. For each profile, it retrieves the password (if available) using `netsh wlan show profile <name> key=clear`.
4. It prints the SSID and password for each profile in a structured format.

## Usage
### Steps to Run the Script:
1. Save the script to a `.py` file, e.g., `wifi_profiles.py`.
2. Open a command prompt or terminal with administrator privileges.
3. Run the script using the command:
   ```bash
   python wifi_profiles.py
   ```
4. The output will display the saved WiFi profiles and their passwords (if any).

### Example Output:
```json
[
  {"ssid": "HomeWiFi", "password": "mypassword123"},
  {"ssid": "OfficeWiFi", "password": "securepass456"}
]
```

## Code Explanation
```python
import subprocess
import re

# Retrieve the output of saved WiFi profiles
command_output = subprocess.run(["netsh", "wlan", "show", "profile"], capture_output=True).stdout.decode()

# Extract profile names using regex
profile_names = (re.findall("all user profile.....:.(.*)\r", command_output))

# List to store WiFi profiles and passwords
wifi_list = list()

# Check if profiles exist
if len(profile_names) != 0:
    for name in profile_names:
        wifi_profile = dict()

        # Retrieve profile information
        profile_info = subprocess.run(["netsh", "wlan", "show", "profile", name], capture_output=True).stdout.decode()

        # Skip profiles without a security key
        if re.search("security key:Absent", profile_info):
            continue
        else:
            wifi_profile["ssid"] = name

            # Retrieve profile password
            profile_info_pass = subprocess.run(["netsh", "wlan", "show", "profile", name, "key=clear"])
            password = re.search("key content(.*)\r", profile_info_pass)

            # Handle cases where no password is available
            if password is None:
                wifi_profile["password"] = None
            else:
                wifi_profile["password"] = password[1]

            wifi_list.append(wifi_profile)

# Print the retrieved WiFi profiles
for x in range(len(wifi_list)):
    print(wifi_list[x])
```

## Notes
- Run this script with **administrator privileges**, as `netsh` commands require elevated permissions to access WiFi profile details.
- Ensure the script is executed in a trusted environment to avoid exposing sensitive information.

## License
This script is free to use and distribute under the MIT License.
