

# **Cron: Complete Guide**

![Cron Schedule](https://img.shields.io/badge/schedule%20%2A%20%2A%20%2A%20%2A%20%2A-black?logo=linux&logoColor=black&color=FCC624)

![node-cron](https://img.shields.io/badge/node%20cron-black?logo=node.js&logoColor=black&color=5fa04e)



## **What is Cron?**

**Cron** is a **time-based job scheduler** used in Unix-like systems to run scripts or commands automatically at specified intervals.

**Key points:**

* Cron is ideal for automation: backups, email notifications, log rotation, DB cleanup, etc.
* It is widely used in **servers, CI/CD pipelines, Node.js apps, and system automation**.

---

## **Cron Syntax**

A standard cron entry has 5 fields (sometimes 6 if seconds are supported) + the command:

```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── Day of the week (0-7, Sunday=0 or 7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of the month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

**Optional 6-field version (Linux, systemd, some Node.js libs):**

```
* * * * * * command_to_run
│ │ │ │ │ │
│ │ │ │ │ │
│ │ │ │ │ └── Day of the week (0-7, Sunday=0 or 7)
│ │ │ │ └──── Month (1-12)
│ │ │ └────── Day of the month (1-31)
│ │ └──────── Hour (0-23)
│ └────────── Minute (0-59)
└─ Seconds (0-59)
```

---

### **Special Characters**

| Symbol     | Meaning                                       |
| ---------- | --------------------------------------------- |
| `*`        | Any value (every unit)                        |
| `,`        | Multiple values (e.g., `1,15` → 1st and 15th) |
| `-`        | Range of values (`1-5` → Mon-Fri)             |
| `/`        | Step values (`*/2` → every 2 units)           |
| `@reboot`  | Run once at system startup                    |
| `@yearly`  | Run once a year (`0 0 1 1 *`)                 |
| `@monthly` | Run once a month (`0 0 1 * *`)                |
| `@weekly`  | Run once a week (`0 0 * * 0`)                 |
| `@daily`   | Run once a day (`0 0 * * *`)                  |
| `@hourly`  | Run every hour (`0 * * * *`)                  |

---

## **Cron Examples (Unix/Linux)**

* **Every minute:**

```bash
* * * * * /home/user/script.sh
```

* **Every 5 minutes:**

```bash
*/5 * * * * /home/user/script.sh
```

* **Every Monday at 6:30 AM:**

```bash
30 6 * * 1 /home/user/weekly_report.sh
```

* **1st of every month at midnight:**

```bash
0 0 1 * * /home/user/monthly_backup.sh
```

* **At server startup:**

```bash
@reboot /home/user/startup.sh
```

---

## **Node.js Cron Scheduling**

### **a) node-cron**

* Lightweight cron scheduler for Node.js.
* Supports **standard 5-field syntax** or **6-field with seconds**.
* Handles **async functions** (with IIFE wrapper).

**Installation:**

```bash
npm i node-cron
```

**Basic Example:**

```js
import cron from "node-cron";

cron.schedule("* * * * *", () => {
  console.log("Runs every minute:", new Date());
});
```

**With Async Function and Error Handling:**

```js
function scheduleTask(sequence, func) {
  cron.schedule(sequence, async () => {
    try {
      await func();
    } catch (err) {
      console.error("Task error:", err);
    }
  });
}

// Usage
scheduleTask("*/5 * * * * *", async () => {
  console.log("Runs every 5 seconds");
});
```

---

### **b) agenda**

* MongoDB-based scheduler for persistent jobs.
* Survives server restarts.
* Can schedule recurring, delayed, or one-time tasks.

```js
import { Agenda } from "agenda";

const agenda = new Agenda({ db: { address: "mongodb://127.0.0.1/agenda" } });

agenda.define("send email", async () => {
  console.log("Email sent at", new Date());
});

(async function() {
  await agenda.start();
  await agenda.every("10 minutes", "send email");
})();
```

---

### **c) bree**

* Uses **worker threads**, good for CPU-heavy tasks.

```js
import Bree from "bree";

const bree = new Bree({
  jobs: [
    { name: "myJob", interval: "5s" } // every 5 seconds
  ]
});

bree.start();
```

---

### **d) later.js**

* Flexible scheduling with **human-readable syntax**:

```js
import later from "later";

later.date.localTime();

const schedule = later.parse.text("every 5 min");
later.setInterval(() => console.log("Runs every 5 minutes"), schedule);
```

---

## **Cron Usage & Best Practices**

1. **Use full paths** in scripts when scheduling via cron.
2. **Log stdout and stderr** for debugging:

```bash
* * * * * /home/user/script.sh >> /home/user/cron.log 2>&1
```

3. **Prevent overlapping tasks** in Node:

```js
let running = false;
cron.schedule("* * * * *", async () => {
  if (running) return;
  running = true;
  await doTask();
  running = false;
});
```

4. **Use environment variables** for database, API keys.
5. **Test locally with seconds-based cron** (`*/5 * * * * *`) before production.

---

## **Node Cron vs Unix Cron**

| Feature     | Unix/Linux Cron                | Node.js node-cron                         |
| ----------- | ------------------------------ | ----------------------------------------- |
| Environment | System-wide                    | Node app process                          |
| Async       | Limited                        | Fully supported (via async IIFE)          |
| Persistence | Automatic (server always runs) | Node process must be alive                |
| Complexity  | Simple shell commands          | JS functions, inline code, advanced logic |


---

## **Summary**

* **Cron** automates repetitive tasks, both on servers (Unix) and inside Node.js apps.
* **Unix cron** is powerful, lightweight, and persistent.
* **Node cron libraries** (`node-cron`, `agenda`, `bree`, `later.js`) make scheduling **inline with code**, **support async**, and integrate with JS apps.
* **Best practices**: logging, full paths, overlapping prevention, error handling, environment variables.
* **Documentation badges** improve clarity in repos and dashboards.

