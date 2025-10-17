
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
* Backend Application that need protection
* Supported regions
  eastus2euap
  centraluseuap
  westcentralus
  eastasia
  uksouth
  northeurope

  ---
  
## 📝 Note

This deployment is intended **for testing purposes only**.  
**Do not use with production workloads.**

---


## ⚙️ Setup Steps

<img width="898" height="1053" alt="image" src="https://github.com/user-attachments/assets/39cd0ce9-ef3d-4ae6-83a7-598816cbac0d" />


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

   | Field                    | Example                        | Description                                                              |
   | ------------------------ | ------------------------------ | ------------------------------------------------------------------------ |
   | **Name**                 | `jwt-validation-demo`          | Friendly name for the validation configuration                           |
   | **Unauthorized Request** | Deny                           | Rejects requests with missing or invalid JWTs                            |
   | **Tenant ID**            | `<your-tenant-id>`             | Must be a valid GUID or one of `common`, `organizations`, or `consumers` |
   | **Client ID**            | `<your-client-id>`             | GUID of the app registered in Entra                                      |
   | **Audiences**            | (Optional) `api://<client-id>` | Expected audience claim. It need to match to scope value                                                 |


4. Scroll down to **Associated routing rules**

   * JWT validation **must be linked to a routing rule** to take effect.
   * Rules must:

     * Be attached to an **HTTPS listener**
     * **Not contain a redirect configuration**

   ✅ If no rules appear, create one first (see below).

---

### ⚙️ Creating an HTTPS Routing Rule for JWT Validation

1. Go to your **Application Gateway → Rules → + Add rule**

2. Configure:

   * **Listener:**

     * Protocol: `HTTPS`
     * Assign certificate or Key Vault secret
   * **Backend target:**

     * Select an existing backend pool or create one
   * **Backend settings:**

     * Use appropriate HTTP/HTTPS port
   * **Rule name:** e.g., `jwt-route-rule`

3. Once created, return to the **JWT Validation Configuration** panel.
   You should now see your HTTPS rule listed under “Associated routing rules”.

4. Check the box next to the rule (e.g., `jwt-route-rule`) and click **Add**.

---

✅ **Result:**
Your JWT validation config is now attached to a secure HTTPS listener and rule.
Only requests that pass JWT validation will be forwarded to the backend pool.

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
Please note - JWT config audience value should match  scope value - https://management.azure.com/.default
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

### 🧠 Notes

* The JWT validation feature requires the **App Gateway Preview** feature flags enabled for portal.
* You can use the same token to authenticate to other Azure resources if the scope is set properly.
* For multi-tenant apps, use `organizations` instead of Tenant GUID.

---

### 📚 References

* [Azure Application Gateway – JWT Validation (Preview)](https://learn.microsoft.com/azure/application-gateway/configuration-overview)
* [Microsoft Entra ID App Registrations](https://learn.microsoft.com/azure/active-directory/develop/quickstart-register-app)

---


