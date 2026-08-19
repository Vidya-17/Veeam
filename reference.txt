Veeam V13 Onboarding and Backup Automation Process
Objective

The objective of this automation was to onboard a Windows server into Veeam Backup & Replication (VBR) v13, deploy the required Veeam components, create a Protection Group, create a Backup Job, execute the backup, and verify a successful backup result.

1. SQL Server Installation

The first step was to install the SQL Server instance that would be used by Veeam Backup & Replication. The installation was automated using Ansible and validated manually. The SQL instance was successfully created and confirmed operational before proceeding with the Veeam installation.

2. VBR Installation

Veeam Backup & Replication v13 was installed silently using the provided installation media and answer file.

During implementation, it was discovered that several service-account and SQL-account parameters inside the answer file were causing validation problems. These entries were removed, and the installation was successfully completed.

After installation, all Veeam services were validated and the VBR console was confirmed accessible.

3. PowerShell 7 Requirement Discovery

While automating Veeam operations through PowerShell, it was discovered that Veeam Backup & Replication v13 requires PowerShell 7 for its PowerShell module.

Initially, Ansible executed commands using Windows PowerShell 5.1, resulting in module loading failures. The solution was to explicitly execute all Veeam PowerShell cmdlets through:

Plain Text
1
C:\Program Files\PowerShell\7\pwsh.exe
Show more lines

This became a standard requirement for all Veeam-related automation tasks.

4. Veeam Deployment Kit (VDK) Generation

The Veeam Deployment Kit (VDK) was generated on the VBR server.

The automation:

Created the deployment directory.
Generated a VDK package containing deployment components.
Created a compressed ZIP archive.
Retrieved the generated package through Ansible.

During troubleshooting, it was discovered that the Veeam Reverse Proxy Service must be running for VDK generation to function correctly.

After correcting this issue, VDK generation completed successfully.

5. VDK Deployment to Target Server

The generated Deployment Kit was transferred to the target Windows server.

The deployment process:

Created the target deployment directory.
Copied the VDK package.
Extracted deployment files.
Executed the Deployment Kit installer.

Several installation failures occurred during testing due to remnants of previous Veeam installations on the target server.

The target was cleaned by removing:

Existing Veeam products
Existing Veeam services
Existing Veeam folders
Existing ProgramData Veeam information

After cleanup, the Deployment Kit installation completed successfully.

6. Protection Group Creation

The next phase was the creation of a Protection Group.

A Protection Group named:

Plain Text
1
Veeam_Target
Show more lines

was created using:

Temporary certificate-based authentication
Individual computer container onboarding

Initially, the playbook reported success but the Protection Group was not actually created.

The root cause was that inline PowerShell execution was not reliably processing all Veeam commands.

The solution was to:

Generate PowerShell scripts on the server.
Execute the scripts using PowerShell 7.

After this modification, the Protection Group was successfully created and visible in the VBR inventory.

7. Protection Group Rescan

Once created, the Protection Group was rescanned.

The rescan process:

Contacted the target server.
Verified connectivity.
Confirmed deployment status.
Registered the server within VBR inventory.

The target machine then became visible under:

Plain Text
1
Physical and Cloud Infrastructure
Show more lines

and was displayed as:

Plain Text
1
Online
2
Backup Agent Installed
Show more lines

confirming successful onboarding.

8. Backup Job Creation

A Windows Computer Backup Job was then created.

Job configuration:

Plain Text
1
Job Name:
2
Veeam_Target_Backup
3
 
4
Backup Type:
5
Entire Computer
6
 
7
Mode:
8
Managed By Backup Server
9
 
10
Platform:
11
Windows
12
 
13
Repository:
14
Default Backup Repository
15
 
16
Retention:
17
7 Days
Show more lines

As with Protection Groups, initial implementations appeared successful but did not actually create jobs.

The same PowerShell 7 script-based approach was then applied.

After correction, the Backup Job was successfully created and validated through PowerShell.

9. Backup Job Execution

The backup job was triggered through automation.

The backup execution process:

Located the created backup job.
Started the backup.
Waited for completion.
Collected session information.

The backup session completed successfully without errors.

10. Backup Verification

After execution, backup sessions were reviewed.

The final backup result showed:

Plain Text
1
Job:
2
Veeam_Target_Backup
3
 
4
Result:
5
Success
6
 
7
State:
8
Stopped
