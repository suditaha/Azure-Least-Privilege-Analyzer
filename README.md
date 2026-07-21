# 🛡️ Azure Least Privilege Analyzer

<img width="652" height="142" alt="Screenshot 2026-07-20 at 9 47 36 PM" src="https://github.com/user-attachments/assets/149e456b-6aca-41b5-a0e5-1b3966638722" />


A CLI tool that audits Azure role assignments across subscriptions and resource groups to identify over-provisioned identities — flagging excessive permissions like Owner or Contributor assigned where Reader would suffice — and produces audit-ready evidence for SOC 2 (CC6.3) and NIST 800-53 (AC-6) compliance.

---

## ✨ Features

- **Automated Evidence Collection:** Scrapes real-time Azure configurations and exports them into timestamped artifacts.
- **Deep RBAC Inspection:** Scans Azure role assignments across subscriptions and resource groups to catch excessive permissions.
- **Least-Privilege Enforcement:** Evaluates identity bindings to flag non-compliant or dangling administrative access.
- **Dual-Format Reporting:** Generates findings in raw JSON and CSV spreadsheets.
- **Pre-Deployment Baselining:** Captures the security state of an environment prior to Infrastructure as Code (IaC) execution.

---

## 🛠️ Technologies Used

- **Cloud Platform:** Microsoft Azure
- **Programming Languages:** Python 3.9+, Bash
- **Infrastructure as Code (IaC):** HashiCorp Terraform
- **APIs & SDKs:** Azure CLI, Azure SDK for Python (`azure-identity`, `azure-mgmt-authorization`, `azure-mgmt-subscription`)
- **Data Processing:** Standard Python libraries (`json`, `csv`)

---

## 🏗️ System Architecture

The architecture relies on a Python-based deep RBAC analyzer and a Bash-driven API scraper for identity metrics.

```text
+-----------------------+        +---------------------------+
|    User / Pipeline    | -----> |  Azure Authentication     |
|   (az login / SPN)    |        | (Azure CLI / Default Cred)|
+-----------------------+        +---------------------------+
           |                                   |
           v                                   v
+-----------------------+        +---------------------------+
|  run_grc_audits.sh    |        | privilege_analyzer.py     |
| (Identity / RBAC CLI) |        | (Deep RBAC Inspection)    |
+-----------------------+        +---------------------------+
           |                                   |
           v                                   v
+------------------------------------------------------------+
|                Azure Resource Manager (ARM)                |
|               (Role Assignments, Identity)                 |
+------------------------------------------------------------+
           |                                   |
           v                                   v
+-----------------------+        +---------------------------+
|    Raw JSON Output    | -----> |      json_to_csv.py       |
|  (Machine Readable)   |        |  (Flattening/Formatting)  |
+-----------------------+        +---------------------------+
           |                                   |
           +-----------------+-----------------+
                             v
               +---------------------------+
               | ./grc-evidence/ (Storage) |
               |   (Timestamped Artifacts) |
               +---------------------------+
```

---

## ⚙️ How It Works

1. **Authentication:** The tool binds to the active Azure session using interactive login credentials or a headless Service Principal (SPN).
2. **Execution:** The Python module queries role assignments via the Azure Authorization API to map users and groups to their role definitions and scopes. The Bash module queries the Azure CLI for identity and access data.
3. **Analysis:** Gathered role assignments are evaluated against a least-privilege baseline, flagging excessive permissions such as Owner or Contributor assigned where Reader would suffice.
4. **Data Transformation:** A Python parser flattens the nested JSON structures returned by Azure into tabular formats.
5. **Evidence Archival:** Timestamped JSON and CSV evidence is saved into the `grc-evidence/` directory, organized by domain.

<img width="1316" height="919" alt="image" src="https://github.com/user-attachments/assets/337d641e-4367-4cd2-958e-ef7caa66dff7" />

---

## 🔒 Security

- **Least Privilege:** The tool itself only needs `Reader` and `Security Reader` access — no risk of accidental infrastructure changes during an audit.
- **Secure Communication:** All communication with Azure Resource Manager is enforced over TLS 1.2+.
- **Secret Management:** No hardcoded credentials. Authentication uses `azure-identity` and environment-based tokens.
- **Audit Trails:** Read-only operations are logged by Azure Activity Logs, and all output is partitioned and timestamped to preserve evidence integrity.
- **Authorization Monitoring:** Identifies authorization drift and highlights excessive access bindings.

---

## 🧠 Key Design Decisions

- **Agentless Execution:** A lightweight script rather than a background daemon — zero operational overhead on target workloads.
- **Dual Output Strategy:** Raw JSON for SIEM integration (e.g. Microsoft Sentinel), CSV for auditors who prefer spreadsheets.
- **Decoupled Architecture:** Separating the deep RBAC analysis (Python) from the identity checks (Bash) allows selective audit execution in pipelines.

---

## 📈 Scalability

- **Stateless Design:** All state is queried dynamically from Azure — safe to run concurrently across multiple subscriptions.
- **Horizontal Scaling:** Deployable as a serverless Azure Function or integrated into CI/CD pipeline stages.
- **API Architecture:** Extraction logic wraps standard REST endpoints, making multicloud expansion straightforward by swapping the API client layer.

---

## 📁 Project Structure

```text
.
├── main.tf                 # Terraform configuration for compliant infrastructure baselines
├── privilege_analyzer.py   # Python Azure SDK engine for deep RBAC scanning
├── run_grc_audits.sh       # Bash entrypoint for Azure CLI identity scraping
├── json_to_csv.py          # Python data transformation engine
├── README.md               # Project documentation
└── grc-evidence/           # Timestamped audit artifacts
    ├── identity/
    ├── pre-deployment/
    └── rbac/
```

---

## 🚀 Installation

1. Clone the repository:
```bash
   git clone <repository-url>
   cd Least_Privilege_Analyzer
```
2. Authenticate with Azure:
```bash
   az login
```
3. Set up the Python environment:
```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install azure-identity azure-mgmt-authorization azure-mgmt-subscription
```
4. Make the execution script executable:
```bash
   chmod +x run_grc_audits.sh
```

---

## 💻 Usage

**Run the deep RBAC analyzer:**
```bash
python privilege_analyzer.py --output-format both
```
Reports are saved to the root directory.

**Run the RBAC and identity audit suite:**
```bash
./run_grc_audits.sh
```
Reports are categorized and saved within `./grc-evidence/`.

---

## 🔮 Future Improvements

- **Multi-Cloud Support:** Extend `privilege_analyzer.py` to parse AWS IAM policies and GCP IAM bindings.
- **Automated Remediation:** Optional commands to automatically revoke non-compliant `Owner` roles via the Azure CLI.
- **CI/CD Integration:** Native GitHub Actions and Azure DevOps pipeline templates for continuous compliance enforcement on pull requests.
- **Agentic AI Analysis:** An AI module to parse generated JSON artifacts and summarize compliance gaps in natural language.

---

## 🎓 Skills Demonstrated

- Azure RBAC and role assignment auditing
- Least-privilege access enforcement and authorization drift detection
- Identity and Access Management (IAM) via Entra ID
- Azure Authorization API integration
- Python SDK and Bash scripting for security automation
- Data engineering — parsing and flattening complex JSON API payloads into relational CSV formats
