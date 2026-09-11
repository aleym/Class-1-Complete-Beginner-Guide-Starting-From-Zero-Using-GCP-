# Class 1 — Complete Beginner Guide (Starting From Zero, Using GCP)

---

## Table of Contents

1. Create a Google Cloud account and a VM
2. Connect to your VM (two ways: browser, or Git Bash)
3. Terminal basics — practice first
4. Set up your project folder
5. Connect your VM to GitHub
6. Get your project onto the server
7. Saving your work with Git
8. Install Docker
9. Run PostgreSQL in Docker
10. Connect visually with DBeaver
11. Connect from Python
12. Design your first tables
13. Quick reference & common errors

---

## 1. Create a Google Cloud account and a VM

1. Go to **console.cloud.google.com** and sign in with (or create) a Google account.
2. If asked, set up a billing account (new accounts get free trial credit — you won't be charged unless you exceed it).
3. In the top search bar, type **Compute Engine** and open it. Click **Enable** if prompted (first time only).
4. Click **VM instances** in the left sidebar → **Create Instance**.
5. Fill in:
   - **Name:** something recognizable, e.g. `my-data-server`
   - **Region/Zone:** pick one close to you
   - **Machine type:** `e2-medium` is a good starting point
   - **Boot disk:** click **Change** → **Public images** → Operating System: **Ubuntu** → Version: **Ubuntu 24.04 LTS** → **Select**
6. Click **Create**. After about a minute, your VM will appear in the VM instances list, running.

---

## 2. Connect to your VM

You have two options. **Start with Option A** — it's the easiest, with zero setup.

### Option A: Browser SSH (easiest — do this first)

1. In the VM instances list, find your server's row.
2. Click the **SSH** button on the right side of that row.
3. A new window opens with a black terminal screen. Wait 10–20 seconds for it to connect.
4. You'll see a prompt like `yourname@instance-name:~$` — **this means you're connected.**

### Option B: Git Bash (Windows — an alternative, once you're comfortable)

**Step 1 — Install Git Bash** (skip if already installed): download from **git-scm.com/download/win**, run the installer, accept the defaults. Open it from your Start menu afterward.

**Step 2 — Generate a key on your laptop** (separate from any key you make on the server itself):
```bash
ssh-keygen -t ed25519
```
Press **Enter** through the prompts to accept defaults.

> **If you see:** `/c/Users/<you>/.ssh/id_ed123456 already exists. Overwrite (y/n)?` — this just means you already made a key before. Type `n` and press Enter to keep the existing one; there's no need to make a new one.

**Step 3 — Display and copy the key:**
```bash
cat ~/.ssh/id_ed25519.pub
```
> **Common typo:** `cat~/.ssh/...` (no space) gives `No such file or directory`. There must be a **space** between `cat` and `~/.ssh/...`.

You'll see one line starting with `ssh-ed25519`. Highlight it with your mouse, then **right-click → Copy** (in Git Bash, Ctrl+C cancels your command instead of copying).

**Step 4 — Add it to your VM:** Google Cloud Console → VM instances → click your instance's **name** → **Edit** → scroll to **SSH Keys** → **Add item** → paste → **Save**.

**Step 5 — Find your VM's external IP:** VM instances list → look at the **External IP** column for your server's row → copy that number.

**Step 6 — Connect:**
```bash
ssh <username>@<external-ip>
```
The username is usually whatever appears right before the `@` at the end of the key line you copied (e.g. if it ends `...you@YourComputer`, your username is `you`). Type `yes` if asked about authenticity the first time.

**Success looks like:** `you@instance-name:~$`

---

## 3. Terminal basics — practice first

Try each of these now, one at a time:

```bash
pwd
```
Shows where you are. First time connecting, this is usually `/home/<you>`.

```bash
ls -la
```
Lists everything in your current folder, including hidden files (anything starting with `.`).

```bash
mkdir test-folder
ls -la
```
Creates a folder, then confirms it now appears in the list.

```bash
cd test-folder
pwd
```
Moves into that folder; `pwd` now shows you're inside it.

```bash
cd ..
pwd
```
Moves back up one level.

```bash
rmdir test-folder
```
Removes the empty practice folder (optional cleanup).

---

## 4. Set up your project folder

```bash
cd /opt
mkdir services
```
> **If you see:** `mkdir: cannot create directory 'services': Permission denied` — this is expected. `/opt` needs administrator permission to write to. Fix:
```bash
sudo mkdir services
```

Now make sure you (not the administrator account) own this folder, so you don't hit permission errors later:
```bash
sudo chown -R $USER:$USER services
cd services
pwd
```
Should print `/opt/services`. This is your project's home from now on.

---

## 5. Connect your VM to GitHub

If you haven't already generated a key **on the server itself** (this is separate from any laptop key from Option B above):
```bash
ssh-keygen -t ed00000
```
Press Enter through all prompts.

Display and copy it:
```bash
cat ~/.ssh/id_ed00000.pub
```

Go to **github.com/settings/keys** → **New SSH key** → paste it in → **Add SSH key**.

Test it:
```bash
ssh -T git@github.com
```
Type `yes` if asked. Success looks like:
```
Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

Tell Git who you are (once per machine):
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

---

## 6. Get your project onto the server

Make sure you're in the right place:
```bash
cd /opt/services
```

**If you already have a GitHub repository:**
```bash
git clone git@github.com:<your-username>/<your-repo>.git
cd <your-repo>
pwd
```

**If you're starting fresh:** on GitHub, click **+** → **New repository** → name it → check **Add a README** → **Create repository**. Then clone it using the command above.

**Lost and not sure where a folder ended up?**
```bash
find / -iname "*your-project-name*" 2>/dev/null
```
This prints the exact full path to anything matching that name, anywhere on the server.

---

## 7. Saving your work with Git

Repeat this cycle every time you make a change:

```bash
git status                    # see what's changed
git add <filename>            # stage a specific file (or "git add ." for everything)
git commit -m "describe it"   # save a snapshot
git push                      # upload to GitHub
git status                    # confirm: "nothing to commit, working tree clean"
```

---

## 8. Install Docker

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verify:
```bash
sudo docker run hello-world
```
Look for: `Hello from Docker! This message shows that your installation appears to be working correctly.`

Run Docker without `sudo`:
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

Start Docker automatically on reboot:
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

## 9. Run PostgreSQL in Docker

Generate a strong password:
```bash
openssl rand -base64 32
```
Copy the output somewhere safe.

Run the database — note the **pinned version** (`postgres:17`, not `postgres:latest`), which avoids a real version-mismatch error that's easy to hit otherwise:
```bash
docker run --rm --name postgres-dev \
  -e POSTGRES_PASSWORD=<paste-your-password> \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  -d postgres:17
```

Confirm it's running:
```bash
docker ps
```

**To let your own laptop (e.g. DBeaver) reach this database from outside the server**, open port 5432 in your VM's firewall:
1. Google Cloud Console → search **Firewall** → VPC network → Firewall.
2. **Create Firewall Rule** → name it `allow-postgres`.
3. Targets: **All instances in the network**. Source IPv4 range: your own IP + `/32` (safer) or `0.0.0.0/0` (testing only).
4. Protocols and ports: **TCP**, port `5432`.
5. **Create**.

---

## 10. Connect visually with DBeaver

1. Download **DBeaver Community Edition** from **dbeaver.io**, install on your own laptop.
2. Open DBeaver → click the plug icon (**New Database Connection**) → **PostgreSQL**.
3. Fill in:
   - **Host:** your VM's external IP
   - **Port:** `5432`
   - **Database:** `mydb`
   - **Username:** `postgres`
   - **Password:** the one you generated
4. Click **Test Connection...** → should succeed. Click **Finish**.

---

## 11. Connect from Python

Back in your SSH session, inside your project folder:

**Create a `.env` file** (holds your real secrets — never uploaded to GitHub):
```bash
nano .env
```
Type:
```
PGHOST=localhost
PGPORT=5432
PGDATABASE=mydb
PGUSER=postgres
PGPASSWORD=<your-generated-password>
```
Save and exit: **Ctrl+O**, **Enter**, **Ctrl+X**.

**Make sure it's ignored by Git:**
```bash
nano .gitignore
```
Type: `.env` — save and exit the same way.

**Set up a virtual environment** (Ubuntu blocks installing packages system-wide by default):
```bash
sudo apt install python3-venv python3-pip
python3 -m venv .venv
source .venv/bin/activate
```
Your prompt should now start with `(.venv)`.

**List your dependencies:**
```bash
nano requirements.txt
```
Type:
```
psycopg2-binary
python-dotenv
```
Save and exit. Install them:
```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

**Write the connection script:**
```bash
nano db_connect.py
```
Paste:
```python
import os
import psycopg2
from dotenv import load_dotenv

load_dotenv()

DB_CONFIG = {
    "host": os.environ.get("PGHOST", "localhost"),
    "port": os.environ.get("PGPORT", "5432"),
    "dbname": os.environ.get("PGDATABASE", "postgres"),
    "user": os.environ.get("PGUSER", "postgres"),
    "password": os.environ.get("PGPASSWORD", ""),
}

def get_connection():
    return psycopg2.connect(**DB_CONFIG)

def main():
    conn = get_connection()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT version();")
            row = cur.fetchone()
            print("Connected to:", row[0])
    finally:
        conn.close()

if __name__ == "__main__":
    main()
```
Save and exit. Run it:
```bash
python db_connect.py
```
You should see: `Connected to: PostgreSQL 17.x ...`

---

## 12. Design your first tables

In DBeaver, open a **SQL Editor** on your connection and run each of these (highlight + Ctrl/Cmd+Enter to execute):

```sql
CREATE SCHEMA IF NOT EXISTS operational;
```

```sql
CREATE TABLE IF NOT EXISTS operational.customers (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(150) NOT NULL,
    email VARCHAR(150),
    registration_date DATE
);
```

```sql
CREATE TABLE IF NOT EXISTS operational.orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date TIMESTAMP NOT NULL
);
```

```sql
ALTER TABLE operational.orders
    ADD CONSTRAINT fk_orders_customer
    FOREIGN KEY (customer_id) REFERENCES operational.customers(customer_id);
```

**Why two tables instead of one?** If one customer can have many orders, cramming everything into one table means repeating the customer's details on every single order row — wasteful, and error-prone if you ever need to correct something. Splitting them and linking via `customer_id` means each piece of information lives in exactly one place. This is the core idea behind **normalisation**.

The **foreign key** (the `ALTER TABLE` above) makes PostgreSQL actively enforce this — it will refuse to create an order for a `customer_id` that doesn't actually exist in `customers`.

---

## 13. Quick reference & common errors

| Command | What it does |
|---|---|
| `ssh <user>@<ip>` | Connect to your server |
| `pwd` | Show where you are |
| `ls -la` | List files, including hidden ones |
| `mkdir <name>` | Create a folder |
| `cd <name>` / `cd ..` | Move into / back out of a folder |
| `nano <file>` | Create/edit a text file (Ctrl+O save, Ctrl+X exit) |
| `git status` | See what's changed |
| `git add . && git commit -m "msg" && git push` | Save and upload a change |
| `docker ps` | See running containers |
| `source .venv/bin/activate` | Turn on your Python environment |
| `exit` | Disconnect from the server |

| Error you might see | What it means | Fix |
|---|---|---|
| `Permission denied` (creating a folder) | You don't own this location | `sudo chown -R $USER:$USER <folder>` |
| `Permission denied (publickey)` | SSH key doesn't match, or `sudo` is being used (which has no key) | Check the key was added correctly; avoid `sudo` before `git` commands |
| `No such file or directory` | Usually a missing space, or wrong path/typo | Re-check exact spacing and spelling |
| `already exists. Overwrite (y/n)?` | A key/file already exists here | Type `n` to keep the existing one, unless you specifically want to replace it |
| `No module named pip` | Python's installer isn't set up | `sudo apt install python3-pip` |
| Docker version/data mismatch error | An old data volume doesn't match a newer image version | Pin a specific version tag (e.g. `postgres:17`) instead of `latest` |
| A password appeared in chat/terminal history | It's now considered exposed | Generate a new one (`openssl rand -base64 32`) and use that instead |

---



That's normal, not a sign you've broken something. Copy-paste the exact command you ran and the exact text that appeared on your screen, and work through it from there — one line at a time, confirming each step before moving to the next.
