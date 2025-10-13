
---

## 🧠 Azure Application Gateway – JWT Validation Setup

![Azure](https://img.shields.io/badge/Azure-blue?logo=microsoftazure\&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-Validation-green)
![Status](https://img.shields.io/badge/Status-Preview-yellow)

This repository demonstrates how to configure **JSON Web Token (JWT) validation** on **Azure Application Gateway** using an **Entra-registered application** and verify token-based access with Azure CLI.

---

### 🚀 Prerequisites

* Azure CLI installed (`az version`)
* Logged in with `az login`
* `jq` installed for JSON parsing
* Contributor or Owner permissions in the target subscription
* Existing Azure Application Gateway

---

## ⚙️ Setup Steps

### 1️⃣ Register an Application in Entra ID

1. Go to [**Azure Portal → App registrations**](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
2. Click **New registration**
3. Enter:

   * **Name:** `appgw-jwt-demo`
   * **Supported account types:** *Accounts in this organizational directory only*
4. Click **Register**
5. Copy the following:

   * **Application (client) ID** → `CLIENT_ID`
   * **Directory (tenant) ID** → `TENANT_ID`

---

### 2️⃣ Create a Client Secret

1. Navigate to **Certificates & Secrets**
2. Click **+ New client secret**
3. Give it a name (e.g. `appgw-secret`)
4. Copy the **Value** (not the Secret ID) → this is your `CLIENT_SECRET`

> ⚠️ Store this securely — it won’t be shown again.

---

### 3️⃣ Configure JWT Validation in Azure Application Gateway

1. Open the preview configuration portal:
   👉 [**App Gateway JWT Config Portal**](https://ms.portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_HybridNetworking=flight23&feature.applicationgatewayjwtvalidation=true)

2. Click **+ Add JWT validation configuration**

3. Fill in:

   | Field                    | Example               | Description                         |
   | ------------------------ | --------------------- | ----------------------------------- |
   | **Name**                 | `jwt-validation-demo` | Friendly name for this config       |
   | **Unauthorized Request** | Deny                  | Reject requests with invalid tokens |
   | **Tenant ID**            | `<your-tenant-id>`    | GUID or `organizations`             |
   | **Client ID**            | `<your-client-id>`    | App registered in Entra             |

4. Click **Add**

---

### 4️⃣ Retrieve Access Token via CLI

```bash
CLIENT_ID="<your-client-id>"
CLIENT_SECRET="<your-client-secret>"
TENANT_ID="<your-tenant-id>"

TOKEN=$(az account get-access-token \
  --service-principal -u "$CLIENT_ID" -p "$CLIENT_SECRET" \
  --tenant "$TENANT_ID" \
  --scope "https://management.azure.com/.default" \
  --query accessToken -o tsv)

echo "$TOKEN"
```

---

### 5️⃣ Verify Token with Application Gateway

Replace values and run:

```bash
curl -k \
  -H "Authorization: Bearer $TOKEN" \
  https://<appgwFrontendIpOrDns>:<listenerPort>/<pathToListenerWithRoute>
```

✅ **Expected Response:**

* **200 OK** → JWT validated successfully
* **401 Unauthorized / 403 Forbidden** → Token invalid or expired

---

### 6️⃣ (Optional) Automate Setup

Use the included [`setup.sh`](./setup.sh) script to automate:

* App registration
* Secret creation
* JWT validation config
* Token generation and verification

#### Run

```bash
chmod +x setup.sh
./setup.sh
```

The script will output:

* Tenant ID
* Client ID
* Token
* Verification results from Application Gateway

---

## 🧾 Outputs Example

```bash
✅ App registered successfully.
Client ID: 5f2e4d5e-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Tenant ID: 72f988bf-xxxx-xxxx-xxxx-xxxxxxxxxxxx
✅ JWT validation configured.
✅ Token retrieved and verified successfully.
```

---

### 🧱 Repository Structure

```
├── setup.sh          # Automates Entra registration + JWT config
├── README.md         # Documentation (this file)
└── examples/
    └── test-curl.sh  # Example verification command
```

---

### 🧠 Notes

* The JWT validation feature requires the **App Gateway Preview** feature flags enabled.
* You can use the same token to authenticate to other Azure resources if the scope is set properly.
* For multi-tenant apps, use `organizations` instead of Tenant GUID.

---

### 📚 References

* [Azure Application Gateway – JWT Validation (Preview)](https://learn.microsoft.com/azure/application-gateway/configuration-overview)
* [Azure CLI – Service Principal Auth](https://learn.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli)
* [Microsoft Entra ID App Registrations](https://learn.microsoft.com/azure/active-directory/develop/quickstart-register-app)

---

Would you like me to include a **diagram (JWT validation flow through AppGW + Entra)** in the README (as an embedded image or ASCII diagram)?
It visually shows how the token travels from client → AppGW → backend for verification.
