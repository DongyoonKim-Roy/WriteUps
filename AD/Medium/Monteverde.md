# Monteverde
# Machine Info
Monteverde is a Medium Windows machine that features Azure AD Connect. The domain is enumerated and a user list is created. Through password spraying, the SABatchJobs service account is found to have the username as a password. Using this service account, it is possible to enumerate SMB Shares on the system, and the $users share is found to be world-readable. An XML file used for an Azure AD account is found within a user folder and contains a password. Due to password reuse, we can connect to the domain controller as mhope using WinRM. Enumeration shows that Azure AD Connect is installed. It is possible to extract the credentials for the account that replicates the directory changes to Azure (in this case the default domain administrator).

# Reconnaissance
