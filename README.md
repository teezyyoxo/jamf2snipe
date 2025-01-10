
# jamf2snipe

## Overview

This tool syncs assets between a Jamf Pro instance and a Snipe-IT instance. It searches for assets based on the serial number rather than the asset tag. If an asset exists in Jamf but not in Snipe-IT, it creates the asset and attempts to match it with an existing Snipe model based on the Mac's model identifier (e.g., `MacBookAir7,1`) being entered as the model number in Snipe. If no match is found, a new model is created.

When syncing, if the asset already exists in Snipe-IT, the tool updates the information specified in the `settings.conf` file and ensures that Jamf's asset tag matches Snipe's, treating Snipe-IT as the authoritative source.

⚠️ **Warning:** If Jamf has blank data, it can overwrite valid data in Snipe-IT. For example, a missing purchase date in JAMF could erase the existing purchase date in Snipe.

If the `asset_tag` in Jamf is blank when creating the asset in Snipe, the tool searches the computer name for a 4+ digit number. If none is found, it assigns `JAMFID-<jamfid#>` as the asset tag in Snipe, making it easy to filter and correct later.


---

## For all intents and purposes...
I run this script on the same Ubuntu server our Snipe site runs on, and our Jamf Pro instance is in the cloud. 
That said, the instructions for Linux are all I can personlly vouch for.

## Requirements

- Python 3.7 or higher with the `requests`, `json`, `time`, and `configparser` libraries installed.
  - Install from [python.org](https://www.python.org/).
  - **Recommended**: Add Python to your PATH. The process for this varies based on environment.
- Network access to both Jamf Pro and Snipe-IT environments.
- Jamf credentials with the following permissions:
 - **Computers:** Read, Update  
 - **Mobile Devices:** Read, Update  
 - **Users:** Read, Update  
- A Snipe-IT API key with create/edit permissions for assets and models.
	- ([How to generate API keys](https://snipe-it.readme.io/reference#generating-api-tokens))

## Installation

### macOS
1. Create a virtual environment:
   ```bash
   mkdir ~/.virtualenv
   python3.X -m venv ~/.virtualenv/jamf2snipe
   source ~/.virtualenv/jamf2snipe/bin/activate`` 

2.  Install dependencies:
    `pip install -r /path/to/jamf2snipe/requirements.txt` 
    
3.  Configure `settings.conf` (copy from `settings.conf.example`).
4.  Run the script: 
    `python jamf2snipe` 

### Linux

1.  Copy the files to `/opt/jamf2snipe/` (recommended) or another directory.
2.  Configure `settings.conf` by copying `settings.conf.example`.
3.  The script checks for `settings.conf` in:
    -   `/opt/jamf2snipe/settings.conf`
    -   `/etc/jamf2snipe/settings.conf`


## Configuration - `settings.conf`

Refer to the [example](https://github.com/grokability/jamf2snipe/blob/main/settings.conf.example) in the documentation.

```
[jamf]
#THESE ARE REQUIRED
- url: https://your_jamf_instance.com:port
- username: Jamf API user username
- password: Jamf API user password

[snipe-it]
#THESE ARE REQUIRED
- url: http://your_snipe_instance.com
- apikey: Snipe-IT API key

**[API mapping]**
# Reference the documentation for Snipe and Jamf to identify all mappable fields, then add them below.
- manufacturer_id: Manufacturer ID for Apple in Snipe-IT
- defaultStatus: Default status ID for new assets
- computer_model_category_id: Category ID for JAMF computers
- mobile_model_category_id: Category ID for JAMF mobile devices
- etc...
```

## Field Mapping (API)
To map fields between Jamf and Snipe-IT, update the `api-mapping` section in `settings.conf`.
**These are some examples:**

-   Computer Name: `name = general name`
-   MAC Address: `_snipeit_mac_address_1 = general mac_address`
-   IPv4 Address: `_snipeit_<your_IPv4_custom_field_id> = general ip_address`
-   Purchase Cost: `purchase_cost = purchasing purchase_price`
-   Purchase Date: `purchase_date = purchasing po_date`
-   OS Version: `_snipeit_<your_OS_version_custom_field_id> = hardware os_version`
-   Extension Attribute: `_snipe_it_<your_custom_field_id> = extension_attributes <attribute id from jamf>`


## Testing
⚠️ **Always test in a non-production environment before full deployment.**

-   Jamf asset tags will sync back to Jamf, but **incorrect configurations can overwrite good data in Snipe-IT**.
-   Spin up a Snipe-IT Docker instance for safe testing.  
    [Snipe-IT Docker Docs](https://snipe-it.readme.io/docs/docker)

## Contributions

Special thanks to the creator of this script, Grokability!
Thanks to all contributors, too! 🙌

1.  Fork this project.
2.  Create a feature branch.
3.  Submit a pull request to the `devel` branch.
4.  Open an issue if necessary and reference it in your PR.

# Changelog
### Version 1.0.9
**Released:** _Jan 10, 2025_
#### 🛠 Changes
-   Enhanced error handling for missing `settings.conf`:
    -   If no valid `settings.conf` is found, the script now:
        -   Prints **recommended locations** for the config file.
        -   Provides a **template** to guide users in creating one.

#### 🔍 Improved Debugging
-   Added clearer logging messages for troubleshooting configuration issues.
#### 💡 User Guidance
-   Improved console output to guide users on how to set up missing configurations.
