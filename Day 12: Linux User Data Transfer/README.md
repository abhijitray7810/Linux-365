# Nautilus App Server 3 - User Data Recovery Guide
![image](https://github.com/abhijitray7810/Linux-365/blob/7aeaa1c91bbd62028f6ca0ad1152a9ad95870727/Day%2012%3A%20Linux%20User%20Data%20Transfer/Screenshot%202026-01-15%20183806.png)
## Task Overview
Locate and copy all files owned by user `mark` from `/home/usersdata` to `/ecommerce` directory while preserving the complete directory structure on App Server 3.

## Problem Statement
- **Issue**: User data accidentally mingled at `/home/usersdata`
- **Objective**: Filter and relocate specific user data
- **User**: mark
- **Source**: `/home/usersdata`
- **Destination**: `/ecommerce`
- **Requirement**: Preserve directory structure, copy files only (exclude directories)

---

## Step-by-Step Implementation

### Step 1: Access App Server 3

SSH into App Server 3:

```bash
# SSH to App Server 3
ssh banner@stapp03

# Enter password when prompted
```

### Step 2: Switch to Root User

```bash
# Switch to root for administrative tasks
sudo su -
```

### Step 3: Verify Source Directory

Check the source directory and verify user `mark` exists:

```bash
# Check if source directory exists
ls -la /home/usersdata

# Verify user 'mark' exists
id mark

# Check ownership of files
ls -lR /home/usersdata | grep mark
```

### Step 4: Create Destination Directory

Ensure the destination directory exists:

```bash
# Create destination directory if it doesn't exist
mkdir -p /ecommerce

# Verify creation
ls -ld /ecommerce
```

### Step 5: Find Files Owned by User 'mark'

Use the `find` command to locate all files owned by `mark`:

```bash
# Find all files (not directories) owned by mark
find /home/usersdata -type f -user mark

# Count the files (optional)
find /home/usersdata -type f -user mark | wc -l
```

### Step 6: Copy Files with Directory Structure Preserved

**Method 1: Using find with cp and --parents (Recommended)**

```bash
# Copy files preserving directory structure
cd /home/usersdata
find . -type f -user mark -exec cp --parents {} /ecommerce \;
```

**Method 2: Using rsync (Alternative)**

```bash
# Using rsync to copy files with structure preserved
rsync -av --include='*/' --include='*' --exclude='*' \
  --prune-empty-dirs \
  $(find /home/usersdata -type f -user mark) \
  /ecommerce/
```

**Method 3: Using tar and find (Alternative)**

```bash
# Create tarball of mark's files and extract to destination
cd /home/usersdata
find . -type f -user mark -print0 | tar --null -czf - -T - | \
  tar -xzf - -C /ecommerce
```

**Method 4: Script-based approach (Most Reliable)**

```bash
# Create a script for copying
cat > /tmp/copy_mark_files.sh << 'EOF'
#!/bin/bash

SOURCE_DIR="/home/usersdata"
DEST_DIR="/ecommerce"
USER="mark"

# Find all files owned by mark
find "$SOURCE_DIR" -type f -user "$USER" | while read -r file; do
    # Get relative path
    rel_path="${file#$SOURCE_DIR/}"
    
    # Create destination directory structure
    dest_file="$DEST_DIR/$rel_path"
    dest_dir=$(dirname "$dest_file")
    
    # Create directory if doesn't exist
    mkdir -p "$dest_dir"
    
    # Copy file preserving permissions and timestamps
    cp -p "$file" "$dest_file"
    
    echo "Copied: $file -> $dest_file"
done

echo "Copy operation completed!"
EOF

# Make script executable
chmod +x /tmp/copy_mark_files.sh

# Run the script
/tmp/copy_mark_files.sh
```

### Step 7: Verify the Copy Operation

Check that files were copied correctly:

```bash
# List files in destination
find /ecommerce -type f

# Count files in destination
find /ecommerce -type f | wc -l

# Compare counts
echo "Source files owned by mark:"
find /home/usersdata -type f -user mark | wc -l
echo "Destination files:"
find /ecommerce -type f | wc -l

# Verify directory structure is preserved
ls -lR /ecommerce

# Check specific file to verify content
# Example: if file exists at /home/usersdata/subdir/file.txt
# It should be at /ecommerce/subdir/file.txt
```

### Step 8: Verify File Ownership and Permissions

```bash
# Check ownership of copied files
ls -lR /ecommerce

# Optionally, change ownership to mark in destination
# (if you want to preserve original ownership)
find /ecommerce -type f -exec chown mark:mark {} \;

# Verify permissions are preserved
stat /home/usersdata/path/to/some/file.txt
stat /ecommerce/path/to/some/file.txt
```

---

## Complete Command Summary

```bash
# 1. SSH to App Server 3
ssh banner@stapp03

# 2. Switch to root
sudo su -

# 3. Verify source directory
ls -la /home/usersdata
id mark

# 4. Create destination directory
mkdir -p /ecommerce

# 5. Copy files preserving directory structure
cd /home/usersdata
find . -type f -user mark -exec cp --parents {} /ecommerce \;

# 6. Verify copy operation
echo "Files owned by mark in source:"
find /home/usersdata -type f -user mark | wc -l
echo "Files in destination:"
find /ecommerce -type f | wc -l

# 7. Check directory structure
ls -lR /ecommerce

# 8. Verify a sample file
find /ecommerce -type f | head -5
```

---

## Detailed Explanation of Key Commands

### Understanding `find` Command

```bash
find /home/usersdata -type f -user mark
```

- `find` - Search for files
- `/home/usersdata` - Starting directory
- `-type f` - Only files (excludes directories)
- `-user mark` - Owned by user 'mark'

### Understanding `cp --parents` Option

```bash
cp --parents source/file.txt /destination
```

- `--parents` - Preserve parent directory structure
- Creates intermediate directories as needed
- Example: `source/dir1/dir2/file.txt` → `/destination/source/dir1/dir2/file.txt`

### Understanding the Script Approach

The script method:
1. Finds each file owned by mark
2. Calculates the relative path from source
3. Creates corresponding directory structure in destination
4. Copies file with permissions (`-p` flag)

---

## Troubleshooting Guide

### Issue 1: Permission Denied Errors

```bash
# Ensure you're running as root
sudo su -

# Check directory permissions
ls -ld /home/usersdata
ls -ld /ecommerce

# Fix permissions if needed
chmod 755 /ecommerce
```

### Issue 2: User 'mark' Doesn't Exist

```bash
# Check if user exists
id mark
getent passwd mark

# If user doesn't exist, check UID instead
# Find files by UID (replace 1001 with actual UID)
find /home/usersdata -type f -uid 1001
```

### Issue 3: Directory Structure Not Preserved

```bash
# If using cp without --parents, directory structure is lost
# Solution: Always use cp --parents or the script method

# Verify you're in the source directory before running
cd /home/usersdata
pwd  # Should show /home/usersdata

# Then run the find command with relative paths
find . -type f -user mark -exec cp --parents {} /ecommerce \;
```

### Issue 4: Destination Directory Already Exists with Files

```bash
# Check existing files
ls -la /ecommerce

# Option 1: Clear destination (BE CAREFUL!)
rm -rf /ecommerce/*

# Option 2: Use different destination
mkdir -p /ecommerce_recovery

# Option 3: Merge (files will be overwritten if same name)
# Use the copy command as normal
```

### Issue 5: No Files Found

```bash
# Verify files exist for user mark
find /home/usersdata -type f -user mark

# If empty, check if files exist at all
ls -lR /home/usersdata

# Check with different user if typo in username
find /home/usersdata -type f -user Mark  # capital M
find /home/usersdata -type f -user marc  # typo
```

### Issue 6: Symbolic Links

```bash
# If there are symbolic links, handle them specially
# Find symbolic links owned by mark
find /home/usersdata -type l -user mark

# Copy symbolic links as-is
find /home/usersdata -type l -user mark -exec cp -P --parents {} /ecommerce \;

# Or copy both files and symlinks
find /home/usersdata \( -type f -o -type l \) -user mark -exec cp --parents {} /ecommerce \;
```

---

## Advanced Options

### Preserve All Attributes

```bash
# Copy with all attributes preserved (timestamps, permissions, ownership)
cd /home/usersdata
find . -type f -user mark -exec cp -a --parents {} /ecommerce \;
```

### Dry Run (Test Before Actual Copy)

```bash
# Preview what will be copied without actually copying
find /home/usersdata -type f -user mark -exec echo "Would copy: {}" \;

# Or create a list file
find /home/usersdata -type f -user mark > /tmp/files_to_copy.txt
cat /tmp/files_to_copy.txt
```

### Copy with Progress Indicator

```bash
# Using rsync with progress
rsync -av --progress --relative \
  $(find /home/usersdata -type f -user mark) \
  /ecommerce/
```

### Create Backup Before Operation

```bash
# Backup existing /ecommerce directory
tar -czf /tmp/ecommerce_backup_$(date +%Y%m%d_%H%M%S).tar.gz /ecommerce

# Verify backup
ls -lh /tmp/ecommerce_backup_*.tar.gz
```

---

## Verification Checklist

- [ ] SSH'd to App Server 3 successfully
- [ ] Switched to root user
- [ ] Verified `/home/usersdata` directory exists
- [ ] Confirmed user `mark` exists on system
- [ ] Created `/ecommerce` destination directory
- [ ] Found files owned by `mark` using find command
- [ ] Copied files preserving directory structure
- [ ] Verified file count matches (source vs destination)
- [ ] Checked directory structure is preserved
- [ ] Verified file permissions/timestamps preserved
- [ ] Tested sample files to ensure content integrity
- [ ] Documented number of files copied

---

## Post-Operation Tasks

### 1. Generate Report

```bash
# Create a report of copied files
cat > /tmp/copy_report.txt << EOF
===================================
File Copy Operation Report
===================================
Date: $(date)
Server: App Server 3 (stapp03)
User: mark
Source: /home/usersdata
Destination: /ecommerce

Files Found in Source:
EOF

find /home/usersdata -type f -user mark >> /tmp/copy_report.txt

cat >> /tmp/copy_report.txt << EOF

Files Copied to Destination:
EOF

find /ecommerce -type f >> /tmp/copy_report.txt

echo "Report saved to /tmp/copy_report.txt"
cat /tmp/copy_report.txt
```

### 2. Verify File Integrity (Optional)

```bash
# Create checksums for verification
cd /home/usersdata
find . -type f -user mark -exec md5sum {} \; > /tmp/source_checksums.txt

cd /ecommerce
find . -type f -exec md5sum {} \; > /tmp/dest_checksums.txt

# Compare checksums
# (Note: paths will differ, so manual comparison needed)
```

### 3. Set Proper Ownership (If Needed)

```bash
# Change ownership of all files in /ecommerce to mark
chown -R mark:mark /ecommerce

# Or keep root ownership
chown -R root:root /ecommerce

# Verify ownership
ls -lR /ecommerce
```

### 4. Set Proper Permissions

```bash
# Set standard permissions
find /ecommerce -type d -exec chmod 755 {} \;
find /ecommerce -type f -exec chmod 644 {} \;

# Or preserve original permissions (if cp -p was used)
# Permissions should already be preserved
```

---

## Testing and Validation

### Test 1: File Count Validation

```bash
# Count and compare
SOURCE_COUNT=$(find /home/usersdata -type f -user mark | wc -l)
DEST_COUNT=$(find /ecommerce -type f | wc -l)

echo "Source files: $SOURCE_COUNT"
echo "Destination files: $DEST_COUNT"

if [ "$SOURCE_COUNT" -eq "$DEST_COUNT" ]; then
    echo "✓ File counts match!"
else
    echo "✗ File counts don't match!"
fi
```

### Test 2: Directory Structure Validation

```bash
# Check if directory structure is preserved
# Pick a sample file and verify its path

# Example: If file exists at /home/usersdata/project/docs/file.txt
# It should exist at /ecommerce/project/docs/file.txt

# Sample test
SAMPLE_SOURCE=$(find /home/usersdata -type f -user mark | head -1)
if [ -n "$SAMPLE_SOURCE" ]; then
    REL_PATH=${SAMPLE_SOURCE#/home/usersdata/}
    DEST_FILE="/ecommerce/$REL_PATH"
    
    echo "Sample source: $SAMPLE_SOURCE"
    echo "Expected destination: $DEST_FILE"
    
    if [ -f "$DEST_FILE" ]; then
        echo "✓ Directory structure preserved!"
    else
        echo "✗ Directory structure not preserved!"
    fi
fi
```

### Test 3: Content Integrity

```bash
# Compare content of a sample file
SAMPLE_SOURCE=$(find /home/usersdata -type f -user mark | head -1)
if [ -n "$SAMPLE_SOURCE" ]; then
    REL_PATH=${SAMPLE_SOURCE#/home/usersdata/}
    DEST_FILE="/ecommerce/$REL_PATH"
    
    # Compare files
    if diff -q "$SAMPLE_SOURCE" "$DEST_FILE" > /dev/null; then
        echo "✓ File content matches!"
    else
        echo "✗ File content differs!"
    fi
fi
```

---

## Common Scenarios and Solutions

### Scenario 1: Large Number of Files

```bash
# For large datasets, use parallel processing
cd /home/usersdata
find . -type f -user mark -print0 | \
  xargs -0 -P 4 -I {} cp --parents {} /ecommerce

# -P 4 means use 4 parallel processes
```

### Scenario 2: Files with Special Characters

```bash
# Handle files with spaces or special characters
cd /home/usersdata
find . -type f -user mark -print0 | \
  xargs -0 -I {} cp --parents {} /ecommerce
```

### Scenario 3: Mixed Ownership in Destination

```bash
# After copy, set specific ownership
chown -R mark:mark /ecommerce

# Or keep current user as owner but mark as group
chown -R root:mark /ecommerce
```

---

## Quick Reference Commands

| Task | Command |
|------|---------|
| Find mark's files | `find /home/usersdata -type f -user mark` |
| Count mark's files | `find /home/usersdata -type f -user mark \| wc -l` |
| Copy with structure | `cd /home/usersdata && find . -type f -user mark -exec cp --parents {} /ecommerce \;` |
| Verify copy | `find /ecommerce -type f \| wc -l` |
| Check ownership | `ls -lR /ecommerce` |
| Change ownership | `chown -R mark:mark /ecommerce` |

---

## Expected Results

After successful execution:

1. **All files owned by mark copied:**
   ```bash
   $ find /home/usersdata -type f -user mark | wc -l
   45  # Example count
   
   $ find /ecommerce -type f | wc -l
   45  # Same count
   ```

2. **Directory structure preserved:**
   ```bash
   # If source has: /home/usersdata/project/docs/file.txt
   # Destination has: /ecommerce/project/docs/file.txt
   ```

3. **Permissions preserved:**
   ```bash
   $ stat /home/usersdata/some/file.txt | grep Access
   Access: (0644/-rw-r--r--)
   
   $ stat /ecommerce/some/file.txt | grep Access
   Access: (0644/-rw-r--r--)
   ```

---

## Cleanup (Optional)

If needed after successful verification:

```bash
# Remove copied files from source (BE VERY CAREFUL!)
# Only do this if task requires moving instead of copying
# find /home/usersdata -type f -user mark -delete

# Create backup before deletion
# tar -czf /tmp/mark_files_backup.tar.gz \
#   $(find /home/usersdata -type f -user mark)
```

---

## Completion Criteria

✅ Connected to App Server 3  
✅ Located all files owned by user `mark` in `/home/usersdata`  
✅ Created destination directory `/ecommerce`  
✅ Copied files preserving directory structure  
✅ Verified file count matches  
✅ Confirmed directory structure is intact  
✅ Validated file permissions preserved  
✅ No directories copied (only files)  

---

**Task Status:** Ready for Implementation  
**Estimated Time:** 10-15 minutes  
**Difficulty:** Intermediate  
**Server:** App Server 3 (Stratos DC)  
**Critical:** Preserve directory structure while copying files only  

---

*Created for: Nautilus Production Support Team*  
*Date: January 15, 2026*  
*Purpose: Data Recovery and Relocation*
