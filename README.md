# Snow Crash

> *"Security isn't about finding vulnerabilities—it's about understanding why they exist."*

**Snow Crash** is an advanced cybersecurity project in the **42** curriculum that introduces a broad range of security concepts through a series of Capture The Flag (CTF) style challenges. Set inside a dedicated Linux virtual machine, each level presents a unique vulnerability or system misconfiguration that must be identified and exploited to progress through the project.

---

## 📚 What This Project Introduced

Throughout the project, I explored multiple areas of computer security and gained practical experience with topics such as:

* 🔐 Linux permissions and privilege escalation
* 🐚 Shell scripting and command injection
* ⚙️ System misconfigurations and insecure services
* 🌐 Network analysis and packet inspection
* 🗂️ File permissions and scheduled tasks
* 🧩 Reverse engineering simple programs
* 💻 Programming and scripting with:

  * **Assembly**
  * **C**
  * **Perl**
  * **PHP**
  * **Lua**

Each challenge introduced a different security concept, demonstrating how seemingly minor implementation mistakes can lead to privilege escalation or unintended access to sensitive information.

---

## 🗂️ Project Structure

Snow Crash is divided into:

| Section   | Levels |
| --------- | :----: |
| Mandatory | **10** |
| Bonus     |  **5** |

Each level revolves around obtaining the privileges of the corresponding `flagXX` user. Once access has been obtained, the `getflag` program reveals a token that serves as the password for the next level, allowing progression through the project until every challenge has been completed.

```bash
$ getflag
Check flag. Here is your token : XXXXXXXXXXXXXXXXXXXX
```

---

## 📁 Repository Layout

Every level in this repository follows the same structure:

```text
levelXX/
├── flag
└── Resources/
```

| File            | Description                                                                 |
| --------------- | --------------------------------------------------------------------------- |
| **flag**        | The recovered token for the next level.                                     |        |
| **Resources**   | Supporting files, notes, scripts, and references used during the challenge. |

---

## 🎯 What I Learned

By completing **Snow Crash**, I developed a stronger understanding of:

* Linux system security
* Privilege escalation techniques
* Exploiting common system misconfigurations
* Basic reverse engineering
* Network and protocol analysis
* Secure programming practices
* Thinking from both an attacker's and a defender's perspective

The project provided a broad introduction to offensive security by exposing vulnerabilities across multiple domains, laying a solid foundation for more advanced topics in binary exploitation and software security.