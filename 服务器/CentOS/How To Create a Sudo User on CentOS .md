# How To Create a Sudo User on CentOS 

### Introduction

The `sudo` command provides a mechanism for granting administrator privileges, ordinarily only available to the root user, to normal users. This guide will show you the easiest way to create a new user with sudo access on CentOS, without having to modify your server's `sudoers` file. If you want to configure sudo for an existing user, simply skip to step 3.

## Steps to Create a New Sudo User

1. Log in to your server as the `root` user.

   ```
   ssh root@server_ip_address

   ```

2. Use the `adduser` command to add a new user to your system.

   Be sure to replace username with the user that you want to create.

   ```
   adduser username

   ```

   - Use the `passwd` command to update the new user's password.

     ```
     passwd username

     ```

   - Set and confirm the new user's password at the prompt. A strong password is highly recommended!

     ```
     Set password prompts:Changing password for user username.
     New password:
     Retype new password:
     passwd: all authentication tokens updated successfully.

     ```

3. Use the `usermod` command to add the user to the `wheel` group.

   ```
   usermod -aG wheel username

   ```

   By default, on CentOS, members of the `wheel` group have sudo privileges.

4. Test sudo access on new user account

   - Use the `su` command to switch to the new user account.

     ```
     su - username

     ```

   - As the new user, verify that you can use sudo by prepending "sudo" to the command that you want to run with superuser privileges.

     ```
     sudo command_to_run

     ```

   - For example, you can list the contents of the `/root` directory, which is normally only accessible to the root user.

     ```
     sudo ls -la /root

     ```

   - The first time you use `sudo` in a session, you will be prompted for the password of the user account. Enter the password to proceed.

     ```
     Output:[sudo] password for username:

     ```

     If your user is in the proper group and you entered the password correctly, the command that you issued with sudo should run with root privileges.