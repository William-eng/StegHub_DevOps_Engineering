Sysadmins can enforce a security policy based upon file permissions. 
---
### All files have three types: 
Owner – Person or process who created the file. 

Group – All users have a primary group, and they own the file, which is useful for sharing files or giving access. 

Others – Users who are not the owner, nor a member of the group.

Also, known as world permission.
We can set the following permissions on both files and directories:
|Permission | File  |Directory
|-----------|-------|---------|
r|Reading access/view file|Users can read file. In other words, they can run the ls command to list contents of the folder/directory.
w| Writing access/update/remove file|Users can update, write and delete a file from the directory.
x|Execution access. Run a file/script as command|Users can execute/run file as command and they have r permission too.
-|No access. When you want to remove r, w, and x permission|All access is taken away or removed.


–rw-r–r– file and drwxr-xr-x directory permission explained
|First character|Description
|---------------|-----------|
-|Regular file.
b|Block special file.
c|Character special file.
d|Directory.
l|Symbolic link.
p|FIFO.
s|Socket.
w|Whiteout.


The chown command changes the user and/or group ownership of for given file. The syntax is: 


          `chown owner-user file 
          chown owner-user:owner-group file
          chown owner-user:owner-group directory
          chown options owner-user:owner-group file`

chmod command: The syntax is:


          `chmod permission file
          chmod permission dir
          chmod UserAccessRightsPermission file `         

          

chmod command: The syntax is:


chmod permission file
chmod permission dir
chmod UserAccessRightsPermission file
