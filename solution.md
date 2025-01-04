---

### **Step 1: Create an Azure Resource Group**
This serves as a container for all resources associated with the project.

```bash
az group create --name BlogRG --location eastus
```

---

### **Step 2: Deploy Azure Static Web Apps**
This hosts the blogging platform's frontend.

1. Ensure you have a GitHub repository for your blogging app (`https://github.com/username/blog-repo`).
2. Deploy the Static Web App using Azure CLI:

```bash
az staticwebapp create \
  --name BloggingPlatform \
  --resource-group BlogRG \
  --location eastus \
  --source https://github.com/username/blog-repo \
  --branch main \
  --app-location "/" \
  --output-location "build"
```

---

### **Step 3: Configure Azure AD B2C for Multi-Tenant Authentication**
1. **Create Azure AD B2C Instance:**
   ```bash
   az ad b2c directory create --name "BlogADDirectory" --location "United States"
   ```
   
2. **Register the Application in Azure AD B2C:**
   - Go to Azure Portal → Azure AD B2C → App registrations → New registration.
   - Add redirect URIs for authentication (e.g., `https://<your-static-app>.azurestaticapps.net`).

3. **Enable Authentication for the Static Web App:**
   ```bash
   az staticwebapp auth update \
     --name BloggingPlatform \
     --resource-group BlogRG \
     --identity-provider AzureActiveDirectory \
     --client-id <YOUR_CLIENT_ID> \
     --issuer-uri <YOUR_B2C_TENANT_ISSUER_URI>
   ```

---

### **Step 4: Create Azure Functions for Background Processing**
Azure Functions handle serverless logic, such as comment moderation and analytics.

1. **Initialize Azure Functions Project:**
   ```bash
   func init BlogFunctions --dotnet
   cd BlogFunctions
   ```

2. **Create Comment Moderation Function:**
   ```bash
   func new --template "HTTP trigger" --name CommentModeration
   ```

3. **Code for Comment Moderation Function:**
   Update the `CommentModeration.cs` file:
   ```csharp
   using System.IO;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.Azure.WebJobs;
   using Microsoft.Azure.WebJobs.Extensions.Http;
   using Microsoft.AspNetCore.Http;
   using Newtonsoft.Json;

   public static class CommentModeration
   {
       [FunctionName("CommentModeration")]
       public static IActionResult Run(
           [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req)
       {
           string requestBody = new StreamReader(req.Body).ReadToEnd();
           var comment = JsonConvert.DeserializeObject<dynamic>(requestBody);

           // Example moderation logic
           bool isApproved = !comment.Content.ToString().Contains("spam");

           return new OkObjectResult(new { Approved = isApproved });
       }
   }
   ```

4. **Deploy Functions to Azure:**
   ```bash
   func azure functionapp publish BlogFunctions
   ```

---

### **Step 5: Set Up Azure Blob Storage for Static Content**
1. **Create a Storage Account:**
   ```bash
   az storage account create --name BlogStorageAccount --resource-group BlogRG --location eastus --sku Standard_LRS
   ```

2. **Configure Blob Storage for Static Content:**
   ```bash
   az storage container create --account-name BlogStorageAccount --name static-content --public-access blob
   ```

---

### **Step 6: Configure Azure CDN**
1. **Create a CDN Profile:**
   ```bash
   az cdn profile create --name BlogCDN --resource-group BlogRG --sku Standard_Microsoft
   ```

2. **Create a CDN Endpoint:**
   ```bash
   az cdn endpoint create \
     --name BlogEndpoint \
     --profile-name BlogCDN \
     --resource-group BlogRG \
     --origin BlogStorageAccount.blob.core.windows.net
   ```

---

### **Step 7: Enable Monitoring with Azure Monitor**
1. **Enable Azure Monitor for Static Web App:**
   ```bash
   az monitor diagnostic-settings create \
     --name BlogMonitor \
     --resource BloggingPlatform \
     --resource-group BlogRG \
     --logs '[{"category": "AppLogs","enabled": true}]' \
     --metrics '[{"category": "AllMetrics","enabled": true}]'
   ```

2. View logs and metrics in the Azure Portal under Azure Monitor.

---

### **Architecture Diagram**

```plaintext
+-----------------+       +----------------+
|   Users         |       | Azure CDN      |
+-----------------+       +----------------+
          |                         |
          v                         v
+-----------------+       +----------------+
| Static Web Apps |       | Azure Blob     |
+-----------------+       | (Static Content)|
          |                         |
          v                         v
+-----------------+       +----------------+
| Azure Functions |       | Azure AD B2C   |
+-----------------+       +----------------+
          |
          v
+-----------------+
| Azure Monitor   |
+-----------------+
```

---

### **Testing the Solution**
1. **Deploy Frontend:** Push your blog frontend code to the linked GitHub repository.
2. **Test Authentication:** Access the static web app and test login via Azure AD B2C.
3. **Submit Comments:** Use an HTTP client like Postman to test the comment moderation function.
4. **Verify CDN:** Load static content via the Azure CDN endpoint.

This project provides a fully scalable, secure, and serverless multi-tenant blogging platform. 
