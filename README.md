# Dynamic DNS

A simple utility to update an existing DNS record on
[Cloudflare](https://cloudflare.com) when the public IP changes.

## Working

The main working is in the [`ddns` script](ddns).

The script gets the public IP from [ipinfo.io/ip](https://ipinfo.io/ip) and
after a rudimentary IP RegEx check, stores it in a file.

The script stores the previously found public IP in a file as well. (Obviously
this file does not exist when the script is run for the first time.)

The script compares the newly found public IP and the previously stored public
IP addresses. If the IP addresses are the same, then it does nothing, but if the
IP addresses don't match, it updates the newly found IP address in the
previously found IP address file and on Cloudflare.

A CRON job runs this script every 10 minutes, so that the DNS record on
Cloudflare always has the current public IP address.

## Usage

-   Clone the repository.

    ```bash
    $ cd ~
    $ git clone https://github.com/HarshKapadia2/dynamic-dns.git
    ```

-   On the Cloudflare dashboard, create a new `A` record with a dummy IP
    address. Turn off Cloudflare's proxying for that DNS record and leave it at
    'DNS Only'.

-   In the repository, create two files `run-ddns` and `list-all-dns-records`,
    add content to it as described in the [`run-ddns`](#run-ddns) and
    [`list-all-dns-records`](#list-all-dns-records) sections, and give them
    executable permissions.

    ```bash
    $ cd ~/dynamic-ddns
    $ vim run-ddns # Add content from the 'run-ddns' section below.
    $ vim list-all-dns-records # Add content from the 'list-all-dns-records' section below.
    $ sudo chmod +x run-ddns
    $ sudo chmod +x list-all-dns-records
    ```

-   Run the `list-all-dns-records` script and make a note of the Record ID of
    the DNS record to be updated. Add that to the `run-ddns` script.

-   Follow the instructions in the ['CRON Script' section](#cron-script) to set
    up a CRON job that will run the `run-ddns` script periodically, so that the
    DNS record on Cloudflare always has the current public IP address.

## Scripts

### `run-ddns`

This script is used to provide sensitive variables to the [`ddns` script](ddns)
and then run it.

> NOTE:
>
> -   [Cloudflare Zone ID](https://developers.cloudflare.com/fundamentals/setup/find-account-and-zone-ids)
> -   [Cloudflare API Token](https://adamtheautomator.com/cloudflare-dynamic-dns/#:~:text=on%20PowerShell%207.1.-,Getting%20a%20the%20Cloudflare%20API%20Token,-When%20updating%20the)
> -   The Cloudflare Record ID can be found by running the [`list-all-dns-records` script](#list-all-dns-records).

```bash
#! /bin/bash

set -Eeuo pipefail

# Set 'dynamic-dns' repository directory path
ddns_repo_path="$(dirname "$0")"

if [[ "${ddns_repo_path}" == "." ]]; then
	ddns_repo_path="$(pwd)"
fi

# Provide environment variables and run `ddns` script
CLOUDFLARE_ZONE_ID="<zone_id_here>" \
	CLOUDFLARE_AUTH_KEY="<api_token_here>" \
	CLOUDFLARE_DNS_RECORD_NAME="<[subdomain.]domainname.tld>" \
	CLOUDFLARE_DNS_RECORD_ID="<id_of_record_to_be_updated_here>" \
	bash "${ddns_repo_path}/ddns"
```

> [Setting environment variables just for one command/script](https://unix.stackexchange.com/questions/495161/import-environment-variables-in-a-bash-script#:~:text=to%20set%20the%20environment%20variable%20just%20for%20this%20script)

### CRON Script

In Ubuntu, this script is to be placed at `/etc/cron.d/dynamic-dns`. It runs a
CRON job every 10 minutes to run the `run-ddns` script.

> NOTE: Replace the `HOME` variable with the path to the `dynamic-dns` repo
> location. Also, replace the `<username>` with the name of the user that should
> run the script (`root` or any other user).

```
# This is to set up a public IP address.
SHELL=/bin/bash
HOME=/home/harsh/dynamic-dns
#
*/10 * * * * <username> ./run-ddns
```

### `list-all-dns-records`

This script lists information about all DNS records. It is used to to get the
Record ID of the DNS record that needs to be updated by the `ddns` script.

> NOTE:
>
> -   [Cloudflare Zone ID](https://developers.cloudflare.com/fundamentals/setup/find-account-and-zone-ids)
> -   [Cloudflare API Token](https://adamtheautomator.com/cloudflare-dynamic-dns/#:~:text=on%20PowerShell%207.1.-,Getting%20a%20the%20Cloudflare%20API%20Token,-When%20updating%20the)

```bash
#! /bin/bash

set -Eeuo pipefail

CLOUDFLARE_ZONE_ID="<zone_id_here>"
CLOUDFLARE_AUTH_KEY="<api_token_here>"

curl --request GET \
	--url "https://api.cloudflare.com/client/v4/zones/${CLOUDFLARE_ZONE_ID}/dns_records" \
	--header "Content-Type: application/json" \
	--header "Authorization: Bearer ${CLOUDFLARE_AUTH_KEY}"
```

## Resources

-   [What Is Dynamic DNS (DDNS), and How Do You Set It Up?](https://www.howtogeek.com/866573/what-is-dynamic-dns-ddns-and-how-do-you-set-it-up)
-   [How to Setup Cloudflare Dynamic DNS](https://adamtheautomator.com/cloudflare-dynamic-dns)
-   CRON Jobs
    -   [Scheduling Cron Jobs with Crontab](https://linuxize.com/post/scheduling-cron-jobs-with-crontab)
    -   [How to Run Cron Jobs Every 5, 10, or 15 Minutes](https://linuxize.com/post/scheduling-cron-jobs-with-crontab)
    -   [Creating a file in `/etc/cron.d`](https://stackoverflow.com/a/64450726/11958552)
    -   [How to view a cron job running currently?](https://stackoverflow.com/a/36883237/11958552)
