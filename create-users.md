Here’s a clean, ready-to-commit Markdown post for your GitHub Pages blog.
Save as e.g. linux-user-management.md (or posts/linux-user-management.md if using a blog structure).

⸻


# Creating and Managing Users in Linux: Interactive, Non-Interactive, and Temporary Accounts

User management in Linux looks simple on the surface, but small flags and fields have important implications—especially in automation, security, and DevOps workflows.

This post covers:
- Interactive vs non-interactive users
- Service accounts
- Temporary users with expiry
- Automation-friendly patterns

---

## 1. Interactive vs Non-Interactive: What It Really Means

People often confuse **two different concepts**.

### A. Non-interactive command execution

A command is non-interactive if it:
- Does not prompt for input
- Can run in scripts, CI/CD, cloud-init

Example:
```bash
useradd -m -s /bin/bash anita
echo "anita:Password" | chpasswd


⸻

B. Non-interactive user account

A user is non-interactive if:
	•	They cannot log in
	•	They cannot get a shell

This is controlled by the login shell, not the password.

⸻

2. How Linux Decides if a User Can Log In

Linux reads /etc/passwd:

username:x:UID:GID:comment:/home/username:SHELL

The last field (SHELL) decides login behavior.

Interactive shells
	•	/bin/bash
	•	/bin/sh
	•	/bin/zsh

Non-interactive shells
	•	/usr/sbin/nologin
	•	/bin/false

⸻

3. Creating an Interactive User

sudo useradd -m -s /bin/bash anita
sudo passwd anita

	•	-m → create home directory
	•	-s → login shell
	•	User can ssh, su -, get a prompt

Verify:

getent passwd anita


⸻

4. Creating a Non-Interactive (Service) User

sudo useradd -m -s /usr/sbin/nologin anita

Why nologin?
	•	Real program (not a shell)
	•	Immediately denies login
	•	Logs the attempt
	•	Safer than /bin/false

Verify:

getent passwd anita
# ...:/home/anita:/usr/sbin/nologin

Test:

su - anita
# This account is currently not available


⸻

5. Understanding id Output

id anita

Example:

uid=1002(anita) gid=1002(anita) groups=1002(anita)

Meaning:
	•	uid → user ID
	•	gid → primary group
	•	groups → supplementary groups

Linux permissions are enforced on numbers, not names.

⸻

6. Making an Existing User Non-Interactive

sudo usermod -s /usr/sbin/nologin anita

Instantly disables login without deleting the user.

⸻

7. Creating a Temporary User (With Expiry)

sudo useradd -m -s /bin/bash -e 2026-01-31 tempuser
sudo passwd tempuser

Verify expiry:

chage -l tempuser

After expiry:
	•	Login denied
	•	User and home directory remain

Disable immediately:

sudo chage -E 0 tempuser

Cleanup:

sudo userdel -r tempuser


⸻

8. Automation-Friendly (Fully Non-Interactive)

For scripts and CI/CD:

sudo -n useradd -m -s /usr/sbin/nologin svcuser
echo "svcuser:Password" | sudo -n chpasswd

	•	-n → fail instead of prompting
	•	Safe for unattended execution

⸻

Key Takeaways
	•	Shell decides interactivity, not the username
	•	/bin/bash → human user
	•	/usr/sbin/nologin → service / system user
	•	useradd is already non-interactive
	•	Expiry is handled via -e or chage

Remember:

Login shell = program executed after login → access or denial

That’s Linux user management in a nutshell.

---

If you want, I can:
- add **front-matter** (date, tags, SEO)
- convert this for **Hugo/Jekyll**
- suggest **blog structure** for GitHub Pages
