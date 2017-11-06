# ssh-rake (SSH Remote Authorized Keys)

Authenticates SSH users via Github keys stored in their profiles. 
Access is granted via Github organization and team. 
ANY SSH key from ANY Github team user will work with ANY user on the server (all or nothing).

At the same time, access via any previously used `authorized_keys` is **BLOCKED**. 
This way, users will have access to the server(s) only if they are granted Github team membership.

Keys are cached locally. The cache is used if API requests start to fail (e.g. Github down).  
There is also a backup SSH key (must be configured), which can be used in case something gets messed up.


## Usage

Download `ssh-rake` script locally and set (hardcode in the script) `GITHUB_TOKEN`, `GITHUB_ORG_NAME`, 
`GITHUB_TEAM_SLUG` and `BACKUP_SSH_PUBLIC_KEY` variables. The script will not work without these configured first.

**IMPORTANT**: All variables are required. If something goes wrong the `BACKUP_SSH_PUBLIC_KEY` is your last resort.

Example:

```
GITHUB_TOKEN='1263a4ee47e5850ca6764e4455b5b7141101cf26'
GITHUB_ORG_NAME='awesome-org'
GITHUB_TEAM_SLUG='ops-team'
BACKUP_SSH_PUBLIC_KEY='ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB...=='
```

`GITHUB_TOKEN` can be generated in your Github settings - https://github.com/settings/tokens. Use scope `read:org`.

You should have SSH access with sudo privileges to the remote server where you plan to install the script.

**IMPORTANT**: log into the server and keep that session open (as a backup in case something goes wrong).

In another console, use the following commands to upload and install the script on the server:

```bash
scp ssh-rake <user>@<host>:/usr/local/bin/ssh-rake
ssh <user>@<host> 'chmod +x /usr/local/bin/ssh-rake && /usr/local/bin/ssh-rake install'
```

Replace `<user>` and `<host>` with the SSH user and SSH host respectively.

Confirm you have access to the server using the backup SSH key:

```bash
ssh -i /path/to/backup-key <user>@<host> 
``` 

Confirm you have access to the server using one of the keys in your Github profile:

```bash
ssh -i /path/to/key <user>@<host> 
``` 

Assuming all of the above worked, you will now be able to give/remove access to your server via the Github team membership.


## Limitations

Github API paging (when there are more than 30 records in a set) is currently not supported.

This means that:
- For organizations with >30 teams you'll have to set `GITHUB_TEAM_ID`
- Only the first 30 users (and their keys) from a team will be accessible

`GITHUB_TEAM_ID` has ot be obtained from the API, but for now there also is an easies way 
(with no guarantees that it will work in the future):

Open the team's edit page (`https://github.com/orgs/<org>/teams/<team>/edit`), view source and find 
`data-alambic-owner-id`. That's the team id.

## TODOs

- Handle API paging (when there are more than 30 records in a set)
- Automated tests
