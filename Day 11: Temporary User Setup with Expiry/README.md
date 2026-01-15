# Nautilus Project - Temporary User Account Setup
![image]()
## Task Overview
Create a temporary user account for developer `ammar` on App Server 3 in the Stratos Datacenter with an expiry date of January 28, 2027.

## Requirements
- **Username**: `ammar` (lowercase)
- **Server**: App Server 3
- **Expiry Date**: 2027-01-28
- **Project**: Nautilus (Temporary Assignment)

---

## Step-by-Step Implementation

### Step 1: Access App Server 3

First, you need to SSH into App Server 3. Check the infrastructure details for connection information.

```bash
# SSH to App Server 3
# Replace with actual server details from infrastructure
ssh <username>@<app-server-3-hostname>

# Or if using a jump host
ssh <username>@<jump-host>
# Then SSH to App Server 3
```

**Common App Server 3 Details:**
- Hostname: Usually `stapp03` or similar
- Username: Check infrastructure details (commonly `banner` for App Server 3)
- Password: Available in infrastructure details

```bash
# Example connection
ssh banner@stapp03
```

### Step 2: Switch to Root User

Once connected, switch to root user to create the account:

```bash
sudo su -
# Or
sudo -i
```

### Step 3: Create User with Expiry Date

Use the `useradd` command with the `-e` flag to set the expiry date:

```bash
# Create user 'ammar' with expiry date 2027-01-28
useradd -e 2027-01-28 ammar
```

**Alternative method (if you prefer more explicit format):**

```bash
# Create user with expiry date in YYYY-MM-DD format
useradd --expiredate 2027-01-28 ammar
```

### Step 4: Set Password for User

Set a password for the new user:

```bash
# Set password for ammar
passwd ammar
```

Enter and confirm the password when prompted.

**For automated/scripted setup:**
```bash
# Set password non-interactively (if needed)
echo "ammar:YourSecurePassword123!" | chpasswd
```

### Step 5: Verify User Creation

Verify that the user was created successfully with the correct expiry date:

```bash
# Check user account details
chage -l ammar
```

**Expected Output:**
```
Last password change                                    : Jan 15, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Jan 28, 2027
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

**Alternative verification methods:**

```bash
# Check /etc/shadow file for expiry date
sudo grep ammar /etc/shadow

# Check user information
id ammar

# List user details
getent passwd ammar

# Verify home directory creation
ls -la /home/ammar
```

### Step 6: Additional User Configuration (Optional)

If needed, configure additional settings:

```bash
# Set user's shell (if not already set)
usermod -s /bin/bash ammar

# Add user to specific groups (if required)
usermod -aG developers ammar

# Create home directory if not auto-created
mkdir -p /home/ammar
chown ammar:ammar /home/ammar
chmod 700 /home/ammar

# Set password aging policies
chage -M 90 ammar  # Password expires every 90 days
chage -W 7 ammar   # Warn 7 days before expiry
```

---

## Complete Command Summary

```bash
# 1. SSH to App Server 3
ssh banner@stapp03

# 2. Switch to root
sudo su -

# 3. Create user with expiry date
useradd -e 2027-01-28 ammar

# 4. Set password
passwd ammar

# 5. Verify user creation
chage -l ammar

# 6. Check user exists
id ammar

# 7. Exit
exit
```

---

## Verification Checklist

- [ ] Successfully SSH'd to App Server 3
- [ ] User `ammar` created in lowercase
- [ ] Expiry date set to 2027-01-28
- [ ] Password configured for the user
- [ ] User account details verified with `chage -l ammar`
- [ ] Account expiry shows "Jan 28, 2027"
- [ ] User can be found in `/etc/passwd`
- [ ] Home directory created (if applicable)

---

## Troubleshooting

### Issue 1: User Already Exists
```bash
# Check if user exists
id ammar

# If exists, delete and recreate
userdel -r ammar
useradd -e 2027-01-28 ammar
```

### Issue 2: Incorrect Date Format
```bash
# Date must be in YYYY-MM-DD format
# Correct: 2027-01-28
# Incorrect: 28-01-2027, 01/28/2027
```

### Issue 3: Permission Denied
```bash
# Ensure you have root privileges
sudo su -

# Or use sudo with commands
sudo useradd -e 2027-01-28 ammar
```

### Issue 4: Verify Expiry Date
```bash
# If expiry date not showing correctly
chage -l ammar | grep "Account expires"

# Manually set expiry date if needed
chage -E 2027-01-28 ammar

# Or use usermod
usermod -e 2027-01-28 ammar
```

### Issue 5: Home Directory Not Created
```bash
# Create home directory manually
mkdir -p /home/ammar
cp -r /etc/skel/. /home/ammar/
chown -R ammar:ammar /home/ammar
chmod 700 /home/ammar
```

---

## Important Commands Reference

### User Management Commands

```bash
# Create user with expiry
useradd -e YYYY-MM-DD username

# Set/change expiry date for existing user
chage -E YYYY-MM-DD username
usermod -e YYYY-MM-DD username

# Remove expiry date
chage -E -1 username

# Check user account status
chage -l username

# Lock user account
usermod -L username

# Unlock user account
usermod -U username

# Delete user
userdel username

# Delete user and home directory
userdel -r username
```

### Verification Commands

```bash
# Check user info
id username
getent passwd username
finger username

# Check shadow file
sudo grep username /etc/shadow

# Check password aging
chage -l username

# List all users
cat /etc/passwd | grep username

# Check user's groups
groups username
```

---

## Expected Results

After successful execution, you should see:

1. **User Created:**
   ```bash
   $ id ammar
   uid=1001(ammar) gid=1001(ammar) groups=1001(ammar)
   ```

2. **Expiry Date Set:**
   ```bash
   $ chage -l ammar | grep "Account expires"
   Account expires                                         : Jan 28, 2027
   ```

3. **User in /etc/passwd:**
   ```bash
   $ grep ammar /etc/passwd
   ammar:x:1001:1001::/home/ammar:/bin/bash
   ```

---

## Security Best Practices

1. **Strong Password**: Ensure password meets complexity requirements
2. **Least Privilege**: Only grant necessary permissions
3. **Audit Trail**: Document user creation in change log
4. **Regular Review**: Monitor temporary accounts before expiry
5. **Automatic Cleanup**: Consider automated scripts to remove expired accounts

---

## Post-Creation Tasks

1. **Notify Developer:**
   - Username: ammar
   - Server: App Server 3
   - Access Duration: Until January 28, 2027
   - Connection Details: [Provide SSH details]

2. **Documentation:**
   - Log the account creation in your CMDB
   - Update access control matrix
   - Set calendar reminder for account expiry

3. **Access Testing:**
   ```bash
   # Test login as ammar
   su - ammar
   # Or from remote
   ssh ammar@stapp03
   ```

---

## Account Expiry Information

**What happens after 2027-01-28?**
- User will not be able to login
- Account becomes locked automatically
- User's files remain on the system
- Admin can extend expiry or delete account

**To extend access later:**
```bash
# Extend expiry to new date
chage -E 2027-03-31 ammar
```

**To remove expiry:**
```bash
# Remove expiry date (permanent account)
chage -E -1 ammar
```

---

## Infrastructure Details

> **Note:** Click on "Details of all Users and Servers" button in the top-right section to get:
> - App Server 3 hostname and IP
> - SSH credentials
> - Network configuration
> - Server specifications

**Typical Stratos Datacenter Structure:**
- **App Server 1**: stapp01
- **App Server 2**: stapp02
- **App Server 3**: stapp03 (Target server)

---

## Quick Reference Card

| Command | Purpose |
|---------|---------|
| `useradd -e 2027-01-28 ammar` | Create user with expiry |
| `passwd ammar` | Set password |
| `chage -l ammar` | View account aging |
| `chage -E 2027-01-28 ammar` | Set/change expiry |
| `id ammar` | Check user exists |
| `userdel -r ammar` | Delete user & home dir |
| `usermod -L ammar` | Lock account |
| `usermod -U ammar` | Unlock account |

---

## Completion Criteria

✅ User `ammar` created on App Server 3  
✅ Username is in lowercase  
✅ Expiry date set to 2027-01-28  
✅ Password configured  
✅ Verification completed with `chage -l ammar`  
✅ Documentation updated  

---

**Task Status:** Ready for Implementation  
**Estimated Time:** 5-10 minutes  
**Difficulty:** Basic  
**Project:** Nautilus Temporary Assignment  

---

*Created for: Stratos Datacenter - App Server 3*  
*Date: January 15, 2026*  
*User: ammar (Temporary Access)*
