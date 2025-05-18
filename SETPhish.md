# Guide: Setting Up Linode VPS with a Domain for Social Engineering Toolkit (SET)

**Important Ethical and Legal Considerations Before You Begin:**

> * **Authorization:** Only use these techniques on systems and networks for which you have explicit, written permission from the owner. Unauthorized access or attacks are illegal and unethical.
> * **Purpose:** This guide is for educational purposes and for individuals conducting authorized penetration tests.
> * **Responsibility:** You are solely responsible for your actions.

**Assumptions:**

* You have an active Linode account.
* You have purchased a domain name from a domain registrar (e.g., GoDaddy, Namecheap, Google Domains). We will use `phishingsite.com` as a placeholder. Make sure it makes sense for your overall goal of the phishing campaign.

---

## Phase 1: Setting Up Your Linode VPS

1.  **Create a New Linode:**
    * Log in to your Linode Cloud Manager.
    * Click on "Create" and select "Linode."
    * **Choose a Distribution:**
        * **Option A (Recommended for ease with SET):** Kali Linux. Kali comes with SET and many other penetration testing tools pre-installed. If available as an image, this can save you setup time.
        * **Option B (More manual but flexible):** A common Linux distribution like Ubuntu 22.04 LTS or Debian 11/12. You'll need to install SET manually. This guide will include steps for this. For this write-up, let's assume you choose **Ubuntu 22.04 LTS**.
    * **Region:** Select a region geographically close to you or your target for lower latency.
    * **Linode Plan:** Choose a plan based on your needs. For basic SET campaigns, a small shared CPU instance (e.g., Nanode 1GB or Linode 2GB) is often sufficient to start. You can resize later if needed.
    * **Label:** Give your Linode a descriptive label (e.g., `set-vps`).
    * **Root Password:** Set a strong root password. You will use this to log in initially.
    * **SSH Keys (Recommended):** Add your SSH public key for more secure, passwordless login.
    * **Add-ons:** You can skip most add-ons for now, like backups, unless you have specific needs (though backups are good practice for any server).
    * Click "Create Linode." Wait for it to provision and boot up.

2.  **Access Your Linode:**
    * Once your Linode is running, find its **IP Address** in the Linode Cloud Manager.
    * Open a terminal (on Linux/macOS) or an SSH client like PuTTY (on Windows).
    * Connect via SSH:
        ```bash
        ssh root@YOUR_LINODE_IP_ADDRESS
        ```
        If you didn't add an SSH key, you'll be prompted for the root password you set.

3.  **Initial Server Setup & Hardening (Ubuntu/Debian Example):**
    * **Update System:**
        ```bash
        sudo apt update && sudo apt upgrade -y
        ```
    * **Create a Non-Root User (Recommended):**
        ```bash
        sudo adduser yourusername
        sudo usermod -aG sudo yourusername
        ```
        Follow the prompts to set a password for the new user.
    * **Configure SSH (Optional but Recommended):**
        * Log out and log back in as your new user: `ssh yourusername@YOUR_LINODE_IP_ADDRESS`
        * Consider disabling root login and password authentication if you've set up SSH keys. Edit `/etc/ssh/sshd_config`:
            ```bash
            sudo nano /etc/ssh/sshd_config
            ```
            Change `PermitRootLogin yes` to `PermitRootLogin no`
          
            Change `PasswordAuthentication yes` to `PasswordAuthentication no` (only if SSH keys are properly set up for `yourusername`)
          
            Save the file and restart SSH: `sudo systemctl restart ssh`
          
            > **Caution:** Ensure you can log in as `yourusername` with sudo privileges *before* disabling root login.
            > 
    * **Configure Firewall (UFW - Uncomplicated Firewall):**
        ```bash
        sudo ufw allow OpenSSH
        sudo ufw allow http  # Port 80
        sudo ufw allow https # Port 443
        sudo ufw enable
        sudo ufw status
        ```
        This allows SSH, HTTP, and HTTPS traffic. You might need to add other rules depending on your SET configuration (e.g., for reverse shells on custom ports).

---

## Phase 2: Domain Name Configuration

1.  **Log in to Your Domain Registrar:** Go to the website where you purchased `phishingsite.com`.
2.  **Navigate to DNS Management:** Find the DNS management section for your domain.
3.  **Create an 'A' Record:**
    * You need to point your domain (`phishingsite.com`) and potentially a subdomain (like `www.phishingsite.com`) to your Linode's IP address.
    * **Record Type:** A
    * **Host/Name:**
        * For `phishingsite.com`: Enter `@` or leave it blank (depends on the registrar).
        * For `www.phishingsite.com`: Enter `www`.
    * **Value/Points to:** Enter `YOUR_LINODE_IP_ADDRESS`.
    * **TTL (Time To Live):** You can usually leave this at the default (e.g., 1 hour or automatic).
    * Save the record(s).

    *Example A Records:*
    | Type | Host/Name | Value/Points to        | TTL     |
    |------|-----------|------------------------|---------|
    | A    | @         | YOUR_LINODE_IP_ADDRESS | Default |
    | A    | www       | YOUR_LINODE_IP_ADDRESS | Default |

4.  **Wait for DNS Propagation:** DNS changes can take anywhere from a few minutes to 48 hours to propagate worldwide, though it's often much faster. You can use online tools like `whatsmydns.net` to check if `phishingsite.com` is resolving to your Linode's IP address.

---

## Phase 3: Installing the Social Engineering Toolkit (SET)

*(Skip this phase if you chose Kali Linux as your Linode distribution, as SET is usually pre-installed. If so, you might just need to update it: `sudo apt update && sudo apt install set` or run the `setoolkit` command and follow update prompts).*

1.  **Install Dependencies (for Ubuntu/Debian):**
    ```bash
    sudo apt update
    sudo apt install git apache2 python3 python3-pip python-is-python3 -y
    # SET might have other specific python library needs,
    # but the installer usually handles them or informs you.
    ```
    (Apache is installed here as it's commonly used by SET for web cloning, but SET can also use its own Python web server).

2.  **Clone SET from GitHub:**
    ```bash
    cd /opt # Or any other directory you prefer, e.g., /usr/local/share
    sudo git clone [https://github.com/trustedsec/social-engineer-toolkit.git](https://github.com/trustedsec/social-engineer-toolkit.git) setoolkit/
    ```

3.  **Run SET Setup:**
    ```bash
    cd setoolkit/
    sudo pip3 install -r requirements.txt
    sudo python3 setup.py install
    ```
    This will install SET and its Python dependencies.

4.  **Launch SET:**
    You can now launch SET by typing:
    ```bash
    sudo setoolkit
    ```

---

## Phase 4: Basic SET Configuration for a Phishing Campaign (Credential Harvester Example)

This is a simplified example. SET has many modules and options.

1.  **Launch SET:**
    ```bash
    sudo setoolkit
    ```

2.  **Navigate the Menus:**
    * You'll see the main menu. Choose `1) Social-Engineering Attacks`.
    * Then choose `2) Website Attack Vectors`.
    * Then choose `3) Credential Harvester Attack`.
    * Then choose `2) Site Cloner`. (You could also use `1) Web Templates` for pre-built pages).

3.  **Configure the Attack:**
    * **IP address for the POST back in Harvester/Tabnabbing:** This should be your Linode's IP address (or `phishingsite.com` if SET resolves it correctly internally, but IP is safer here). Enter `YOUR_LINODE_IP_ADDRESS`.
    * **Enter the URL to clone:** Enter the full URL of the legitimate website you have *permission* to clone for your test (e.g., `https://example-target-login.com`).
    * SET will then ask if you want to start the SET web server or use Apache.
        * If you choose SET's web server (often default), it will typically run on port 80.
        * If you have Apache running and configured it, you might choose that. For simplicity with default SET behavior, let SET manage its web server.

4.  **Accessing the Phishing Site:**
    * Once SET starts the web server and cloning process, it will tell you that the harvester is running.
    * Now, if your DNS for `phishingsite.com` has propagated to `YOUR_LINODE_IP_ADDRESS`, anyone Browse to `http://phishingsite.com` (or `http://www.phishingsite.com`) should see the cloned website.
    * Any credentials entered on this fake page will be captured by SET and displayed in your terminal, and typically saved to a log file in the SET output directory (usually under `/root/.set/reports/` or a similar path shown by SET).

---

## Phase 5: Deployment Considerations & Best Practices

1.  **SSL/TLS Certificate (Highly Recommended):**
    * Most users are wary of HTTP sites, especially for logins. A phishing site without HTTPS (no padlock) is much less believable.
    * You can get a free SSL/TLS certificate from Let's Encrypt.
    * **Using Certbot with Apache/Nginx:** If you decide to use Apache or Nginx as your primary web server (instead of or in conjunction with SET's built-in server), Certbot makes this easy:
        ```bash
        # For Apache
        sudo apt install certbot python3-certbot-apache
        sudo certbot --apache

        # For Nginx
        # sudo apt install certbot python3-certbot-nginx
        # sudo certbot --nginx
        ```
        Follow the prompts, providing `phishingsite.com` and `www.phishingsite.com` when asked.
    * **SET and SSL:** SET has some capabilities to work with SSL, or you might configure a reverse proxy (like Nginx or Apache) to handle SSL and forward traffic to SET's web server. This is a more advanced setup.
    * Ensure your firewall (UFW) allows HTTPS traffic on port 443.

2.  **Stopping SET and Web Services:**
    * In the SET terminal, you can usually stop the attack/server with `Ctrl+C`.
    * If you used Apache, you could stop it with `sudo systemctl stop apache2`.

3.  **Log Monitoring and Cleanup:**
    * SET stores reports, including captured credentials. Be sure to securely handle and delete this data after your authorized engagement according to your agreement.
    * Regularly check server logs (`/var/log/`) for any unusual activity.

4.  **Linode Backups:**
    * Consider enabling Linode's backup service for your VPS if you plan to reuse or maintain this setup for an extended period.

5.  **Domain Reputation:**
    * Be aware that using a domain for phishing (even authorized) can quickly get it flagged by security vendors and blacklisted. This is expected but something to be mindful of for future use of that specific domain. For legitimate pentesting, this is part of the process.

---

**Disclaimer (Reiteration):**

> * This guide is for educational and *authorized ethical testing purposes only*.
> * The specific commands and menu options in SET or Linode's interface may change over time. Refer to official documentation if you encounter discrepancies.

You should now have a foundational understanding of how to set up a Linode VPS with a domain for use with the Social Engineering Toolkit. Remember to proceed responsibly and ethically.
