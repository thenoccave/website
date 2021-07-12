# The WinRM client received an HTTP server error status FIX
We recently had an issue with a client when attempting to access Outlook Web Access they received a blank page however the server was returning a 500 status. Attempting to access the Exchange Console the below error appeared.
![](/img/Exchange-Kerberos-auth-failed.png)

```
The following error occured while attempting to connect to the specified Exchange server "server name":
 
The attempt to connect to &amp;amp;amp;amp;amp;lt;server address&amp;amp;amp;amp;amp;gt;/PowerShell using "Kerberos" authentication failed: Conencting to the remote server failed with the following error message: The WinRM client received an HTTP server error status (500), but the remote service did not include any other information about the cause of the failure. For more information, see the about_Remote_Troubleshooting Help topic.
```
In the event log the following:
![](/img/Exchange-Kerberos-auth-failed-2.png)

The errors were
```
ASP.NET 2.0.50727.0
 
Failed to initialize the AppDomain:/LM/W3SVC/1/ROOT/owa
 
Exception: System.SystemException
 
Message: Failed to create AppDomain.
 
StackTrace:    at System.Web.Hosting.ApplicationManager.CreateAppDomainWithHostingEnvironment(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
   at System.Web.Hosting.ApplicationManager.CreateAppDomainWithHostingEnvironmentAndReportErrors(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
 
InnerException: System.Security.XmlSyntaxException
 
Message: Invalid syntax.
 
StackTrace:    at System.Security.SecurityDocument.InternalGetElement(Int32&amp;amp;amp;amp;amp; position, Boolean bCreate)
   at System.Security.SecurityDocument.InternalGetElement(Int32&amp;amp;amp;amp;amp; position, Boolean bCreate)
   at System.Security.SecurityDocument.GetChildrenPositionForElement(Int32 position)
   at System.Security.Policy.PolicyStatement.FromXml(SecurityDocument doc, Int32 position, PolicyLevel level, Boolean allowInternalOnly)
   at System.Security.Policy.PolicyLevel.CheckCache(Int32 count, Char[] serializedEvidence)
   at System.Security.Policy.PolicyLevel.Resolve(Evidence evidence, Int32 count, Char[] serializedEvidence)
   at System.Security.PolicyManager.CodeGroupResolve(Evidence evidence, Boolean systemPolicy)
   at System.Security.PolicyManager.ResolveHelper(Evidence evidence)
   at System.Security.PolicyManager.Resolve(Evidence evidence)
   at System.Security.SecurityManager.ResolvePolicy(Evidence evidence, PermissionSet reqdPset, PermissionSet optPset, PermissionSet denyPset, PermissionSet&amp;amp;amp;amp;amp; denied, Boolean checkExecutionPermission)
   at System.Security.SecurityManager.ResolvePolicy(Evidence evidence, PermissionSet reqdPset, PermissionSet optPset, PermissionSet denyPset, PermissionSet&amp;amp;amp;amp;amp; denied, Int32&amp;amp;amp;amp;amp; securitySpecialFlags, Boolean checkExecutionPermission)
   at System.AppDomain.nSetupDomainSecurity(Evidence appDomainEvidence, IntPtr creatorsSecurityDescriptor, Boolean publishAppDomain)
   at System.AppDomain.SetDomainManager(Evidence providedSecurityInfo, Evidence creatorsSecurityInfo, IntPtr parentSecurityDescriptor, Boolean publishAppDomain)
   at System.AppDomain.InternalRemotelySetupRemoteDomainHelper(Object[] args)
   at System.AppDomain.nCreateDomain(String friendlyName, AppDomainSetup setup, Evidence providedSecurityInfo, Evidence creatorsSecurityInfo, IntPtr parentSecurityDescriptor)
   at System.AppDomain.CreateDomain(String friendlyName, Evidence securityInfo, AppDomainSetup info)
   at System.Web.Hosting.ApplicationManager.CreateAppDomainWithHostingEnvironment(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
 
An error occurred while trying to start an integrated application instance.
 
Exception: System.SystemException
 
Message: Failed to create AppDomain.
 
StackTrace:    at System.Web.Hosting.ApplicationManager.CreateAppDomainWithHostingEnvironment(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
   at System.Web.Hosting.ApplicationManager.CreateAppDomainWithHostingEnvironmentAndReportErrors(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
   at System.Web.Hosting.ApplicationManager.GetAppDomainWithHostingEnvironment(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
   at System.Web.Hosting.ApplicationManager.CreateObjectInternal(String appId, Type type, IApplicationHost appHost, Boolean failIfExists, HostingEnvironmentParameters hostingParameters)
   at System.Web.Hosting.ProcessHost.StartApplication(String appId, String appPath, Object&amp;amp;amp;amp;amp; runtimeInterface)
 
InnerException: System.Security.XmlSyntaxException
 
Message: Invalid syntax.
 
StackTrace:    at System.Security.SecurityDocument.InternalGetElement(Int32&amp;amp;amp;amp;amp; position, Boolean bCreate)
   at System.Security.SecurityDocument.InternalGetElement(Int32&amp;amp;amp;amp;amp; position, Boolean bCreate)
   at System.Security.SecurityDocument.GetChildrenPositionForElement(Int32 position)
   at System.Security.Policy.PolicyStatement.FromXml(SecurityDocument doc, Int32 position, PolicyLevel level, Boolean allowInternalOnly)
   at System.Security.Policy.PolicyLevel.CheckCache(Int32 count, Char[] serializedEvidence)
   at System.Security.Policy.PolicyLevel.Resolve(Evidence evidence, Int32 count, Char[] serializedEvidence)
   at System.Security.PolicyManager.CodeGroupResolve(Evidence evidence, Boolean systemPolicy)
   at System.Security.PolicyManager.ResolveHelper(Evidence evidence)
   at System.Security.PolicyManager.Resolve(Evidence evidence)
   at System.Security.SecurityManager.ResolvePolicy(Evidence evidence, PermissionSet reqdPset, PermissionSet optPset, PermissionSet denyPset, PermissionSet&amp;amp;amp;amp;amp; denied, Boolean checkExecutionPermission)
   at System.Security.SecurityManager.ResolvePolicy(Evidence evidence, PermissionSet reqdPset, PermissionSet optPset, PermissionSet denyPset, PermissionSet&amp;amp;amp;amp;amp; denied, Int32&amp;amp;amp;amp;amp; securitySpecialFlags, Boolean checkExecutionPermission)
   at System.AppDomain.nSetupDomainSecurity(Evidence appDomainEvidence, IntPtr creatorsSecurityDescriptor, Boolean publishAppDomain)
   at System.AppDomain.SetDomainManager(Evidence providedSecurityInfo, Evidence creatorsSecurityInfo, IntPtr parentSecurityDescriptor, Boolean publishAppDomain)
   at System.AppDomain.InternalRemotelySetupRemoteDomainHelper(Object[] args)
   at System.AppDomain.nCreateDomain(String friendlyName, AppDomainSetup setup, Evidence providedSecurityInfo, Evidence creatorsSecurityInfo, IntPtr parentSecurityDescriptor)
   at System.AppDomain.CreateDomain(String friendlyName, Evidence securityInfo, AppDomainSetup info)
   at System.Web.Hosting.ApplicationManager.CreateAppDomainWithHostingEnvironment(String appId, IApplicationHost appHost, HostingEnvironmentParameters hostingParameters)
 
An application has reported as being unhealthy. The worker process will now request a recycle. Reason given: ASP.NET application initialization failed.. The data is the error.
```
The solution is actually rather simple. Rename the file:
```
C:\Windows\Microsoft.NET\Framework64\v2.0.50727\CONFIG\enterprisesec.config
```

Just add a .bak or something to the end. Everything should start working again, no need to reset any services