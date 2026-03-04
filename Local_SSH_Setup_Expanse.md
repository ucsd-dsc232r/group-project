# Setting Up Local SSH & TOTP Access for SDSC Expanse

This setup is optional but to connect to the SDSC Expanse cluster directly from your local terminal, you need to use SSH keys combined with a Time-Based One-Time Password (TOTP). Password-only SSH access is not supported.

Follow these steps carefully to configure your local machine.

---

## Step 1: Find Your Official SDSC Username
Your SSH username is **not** your standard university email address. 

1. Log into the [SDSC Expanse Portal](https://portal.expanse.sdsc.edu/).
2. Open the **"expanse Shell Access"** app to launch a web terminal.
3. Type `whoami` and press Enter to see your exact SDSC username. Save this for Step 5.

## Step 2: Get Your Local Public SSH Key
1. Open your local computer's terminal (Terminal on Mac/Linux, PowerShell or WSL on Windows).
2. Check if you already have a key by running:
   ```bash
   cat ~/.ssh/id_rsa.pub
   # OR
   cat ~/.ssh/id_ed25519.pub
   ```
3. Copy the entire output string (it usually starts with `ssh-rsa` or `ssh-ed25519`). 
*(Note: If you get an error saying the file doesn't exist, you need to generate a key first by running `ssh-keygen` and pressing Enter through all the default prompts).*

## Step 3: Add Your Key to Expanse
You must append your local public key to your authorized keys file on the Expanse cluster.

1. Go back to the "expanse Shell Access" web terminal on the SDSC portal.
2. Open your authorized keys file by running:
   ```bash
   mkdir -p ~/.ssh
   touch ~/.ssh/authorized_keys
   nano ~/.ssh/authorized_keys #OR vim ~/.ssh/authorized_keys
   ```
3. Paste the public key string you copied in Step 2 onto a new line in this file. 
4. Save and exit (In nano: `Ctrl+O`, `Enter`, `Ctrl+X`; In Vim: `:wq`).


## Step 4: Enroll Your Device for TOTP (2FA)
Expanse requires a secondary authenticator app for external terminal access.

1. Go to the SDSC Passive Portal: [https://passive.sdsc.edu](https://passive.sdsc.edu).
2. Click **Login using Globus** and select **ACCESS CI** as your organization.
3. Click the **Manage 2FA** button.
4. Scan the provided QR code with your authenticator app (Duo Mobile, Google Authenticator, Authy, etc.).

> **IMPORTANT:** It can take up to 15 minutes for the Expanse servers to sync your new TOTP enrollment across all nodes. Please wait before proceeding to the final step.

## Step 5: Connect via SSH
Once the sync is complete, you are ready to log in from your local machine!

1. Open your local computer's terminal.
2. Run the SSH command using the hostname `login.expanse.sdsc.edu` and your specific username:
   ```bash
   ssh <your_sdsc_username>@login.expanse.sdsc.edu
   ```
3. Because your SSH key is authorized, it will skip the password prompt.
4. You will immediately be asked for the Verification Code. Enter the 6-digit TOTP code from your authenticator app. 

You should now be successfully connected to the Expanse login node directly from your terminal!

## References
https://www.sdsc.edu/systems/expanse/user_guide.html
