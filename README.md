# JWTValidation
Azure Application Gateway steps to validate JWT token 
Got it — here’s exactly how you can complete those GitHub setup steps and configure JWT validation with Azure Application Gateway:

---

### 🧩 **Step 1 — Register an App in Microsoft Entra ID**

1. Go to the [Azure Portal → Entra ID → App registrations](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
2. Click **“+ New registration.”**
3. Enter:

   * **Name:** e.g., `appgw-jwt-demo`
   * **Supported account types:** Choose “Accounts in this organizational directory only.”
   * **Redirect URI (optional):** Leave blank or set to your App Gateway frontend URL (optional at this step).
4. Click **Register.**

After registration:

* Copy the **Application (Client) ID** → this is your **CLIENT_ID**.
* Copy the **Directory (Tenant) ID** → this is your **TENANT_ID**.

---

### 🔑 **Step 2 — Create a Client Secret**

1. Under the app you just created → go to **Certificates & secrets**.
2. Click **“+ New client secret.”**
3. Give it a name and choose expiration.
4. Copy the **Value** (not the Secret ID) — this is your **CLIENT_SECRET**.

Store it securely — it won’t be visible again.

---

### ⚙️ **Step 3 — Configure JWT Validation in Azure Application Gateway**

1. Open this link directly:
   👉 [JWT Validation Config Portal](https://ms.portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_HybridNetworking=flight23&feature.applicationgatewayjwtvalidation=true)
2. Click **“+ Add JWT validation configuration.”**
3. Fill in:

   * **Name:** e.g., `jwt-validation-demo`
   * **Unauthorized Request:** Deny
   * **Tenant ID:** your Tenant ID (GUID or `organizations` if multi-tenant)
   * **Client ID:** your registered App Client ID (GUID)
   * **Audience (optional):** leave blank or specify the same Client ID
4. Click **Add.**

---

### 🧠 **Step 4 — Retrieve Access Token using Azure CLI**

Run this in your terminal:

```bash
CLIENT_ID="<your-client-id>"
CLIENT_SECRET="<your-client-secret>"
TENANT_ID="<your-tenant-id>"

TOKEN=$(az account get-access-token \
  --service-principal -u "$CLIENT_ID" -p "$CLIENT_SECRET" --tenant "$TENANT_ID" \
  --scope "https://management.azure.com/.default" \
  --query accessToken -o tsv)
```

You can verify the token by printing:

```bash
echo "$TOKEN"
```

---

### 🌐 **Step 5 — Test JWT Validation via Application Gateway**

Use `curl` to send a test request with the token:

```bash
curl -k \
  -H "Authorization: Bearer $TOKEN" \
  https://<appgwFrontendIpOrDns>:<listenerPort>/<pathToListenerWithRoute>
```

If configured correctly, you should receive a **200 OK** response from the backend (authorized).
If JWT validation fails, you’ll receive a **401 Unauthorized** or **403 Forbidden** response.

---

Would you like me to generate a **`setup.sh`** script (automated version of these steps) that you can include directly in your GitHub repo? It can:

* Register the app via Azure CLI
* Capture the Tenant/Client IDs
* Create the secret
* Configure the App Gateway automatically.

