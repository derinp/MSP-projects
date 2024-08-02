## Objective
note: i will be using generic names such as Server001 VM001 PC001 to keep client information confidential.

Ive been tasked to do a live migration of a management VM which lives on a HV server. Scope of work is moving it from Server001 to Server002. Both of these servers are onsite at the client location. This is a non prod server,
this means ill be able to shut the VM down without causing issues. The main point is to migrate this WITHOUT having to shut the VM down unless absolutley necessary. 

Ill be leaning heavily on microsofts documentation to get this done: https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/live-migration-overview

## Migration Time!
Logging on to the Server that hosts both HVs, we can see both of them up and running.

![01-HVs](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/01-HVs.png)

We can also see the MGMT Vm that needs to be moved over:

![02-mgmt](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/02-mgmt.png)

First we need to verify that both HV001 and HV002 are on the same trusted domain. Logging into both, we run an "echo %USERDOMAIN%" command and verify they are on the same domain.

Heading over to the domain controller, there are some settings within the delegation tab we need to confirm (HV001>right click>properties>Delegation).
We must make sure the highlights are selected and 2 specific services running: 
1. cifs
2. Microsoft Virtual System Migration
   
![03-hvproperties](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/03-hvproperties.png)

i confirmed all necessary services were running on both HVs.

Next, we have to verify some settings within the Hyper-V Manager. Righ clicking HV001 > settings > Live migrations, i check the "Enable incoming and outgoing live migrations" box. Under Incoming live migrations, i verify "Use any available network for live migration" is checked.

![04-livemigrationsettings](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/04-livemigrationsettings.png)

Under Advanced settings, make sure that "User Kerberos" is selected

![05-kerberos](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/05-kerberos.png)

i verify the same settings are configured on both hosts.

Right clicking the MGMT server and selecting Move, a wizard pops up.

1. first step is "Before you being" simply read over it and select Next.
2. Second step "Choose Move Type," i select Move Virtual Mahcine
3. Third step "Specify Destination" i want to move this to HV002. Enter the Host name accordingly. Hit Next
4. Fourth step "Choose move options" I select Move the virtual machine's data to a single location. Hit Next
5. Fifth step "Virtual Machine" we are asked to select a desination location. I select the appropriate destination. Hit next
6. Sixth step "Summary" review and verify everything looks good and hit "finish"

I ended up running into an issue:

![06-error](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/06-error.png)

immediately the fix wasnt apparent to me. After searching up the error and reading through a few community articles, I was able to pinpoint the solution.

![07-fix](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/07-fix.png)

Unfortunetly, we had to power down the machine.... so not a complete live migration. We select the option for processor compatability and try to migrate again.
Successfully done!

![08-done](https://github.com/derinp/MSP-projects/blob/main/Live%20Migrations/08-done.png)
