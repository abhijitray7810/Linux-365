I'll help you create the user 'james' on App Server 3 without a home directory. Let me first check the infrastructure details to understand the server setup.
Based on the infrastructure details, I can see that App Server 3 (stapp03) has:
- IP: 172.16.238.12
- Hostname: stapp03.stratos.xfusioncorp.com
- User: banner
- Password: BigGr33n

To create the user 'james' without a home directory, you'll need to SSH into App Server 3 and use the `useradd` command with the appropriate options.

**Steps to create user 'james' without home directory:**

```bash
# SSH into App Server 3
ssh banner@stapp03.stratos.xfusioncorp.com
# Password: BigGr33n

# Create user without home directory
sudo useradd -M james

# Or alternatively
sudo useradd --no-create-home james

# Set password for the user (if needed)
sudo passwd james
```

**Key points:**
- `-M` or `--no-create-home` flag prevents home directory creation
- The user will be created with default shell (usually `/bin/bash`)
- No home directory will exist at `/home/james`
- The user account will be functional for login but won't have a personal directory

**Verification:**
```bash
# Check if user exists
grep james /etc/passwd

# Verify no home directory was created
ls -la /home/ | grep james
```
![image](https://github.com/abhijitray7810/Linux-365/blob/3e9902d9450054c7c8f56a77b13bf0c4660999fc/Day%2010%3A%20Service%20User%20Creation%20without%20Home%20Directory/Screenshot%202026-01-15%20164313.png)
The user 'james' is now created on App Server 3 without a home directory as requested.
