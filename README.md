# Multi-Tenant-Blogging-Platform

### **Project : Multi-Tenant Blogging Platform**

#### **Client Requirement:**
- Host a multi-tenant blogging platform.
- Provide serverless scalability for unpredictable traffic.
- Use Azure Functions for background processing.
- Secure user data with Azure AD B2C.
- Implement CDN for faster content delivery.

---

#### **Solution Overview:**
1. **Hosting:**  
   Deploy the blogging application on Azure Static Web Apps.
2. **Serverless Functions:**  
   Use Azure Functions to handle comment moderation and analytics.
3. **Authentication:**  
   Configure Azure AD B2C for tenant-based user authentication.
4. **Content Delivery:**  
   Set up Azure CDN for static content.
5. **Logging & Monitoring:**  
   Enable Azure Monitor for logs and performance metrics.

---

#### **Code Snippet:**
```bash
# Deploy Static Web App
az staticwebapp create --name BloggingPlatform --resource-group BlogRG --location eastus \
    --source https://github.com/username/blog-repo --branch main --app-location "/" --output-location "build"

# Configure Azure Functions
func init BlogFunctions --dotnet
func new --template "HTTP trigger" --name CommentModeration
func azure functionapp publish BlogFunctions

# Set up Azure CDN
az cdn profile create --name BlogCDN --resource-group BlogRG --sku Standard_Microsoft
az cdn endpoint create --name BlogEndpoint --profile-name BlogCDN --resource-group BlogRG --origin BlogStorageAccount.blob.core.windows.net
```

---

#### **Architecture Diagram:**

- Users → Azure CDN → Azure Static Web Apps  
- Azure Functions (background processing)  
- Azure AD B2C (authentication)  
- Azure Blob Storage (static files)  
- Azure Monitor (performance and logs)  

---
