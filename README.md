# AKM-Docs
Documentation for AKM
---
title: AKM Key Connection for SQL Server User Guide
permalink: /akm_key_connection_for_sql_server_user_guide/
last_updated: 06.28.2018
sidebar: akm_sidebar
folder: akm
---

# Chapter 1: About This Manual

## Key Connection for SQL Server

Townsend Security’s Key Connection for SQL Server (Key Connection) is a ready to use client application which integrates AKM with Microsoft SQL Server 2008 and later.

## Who is this for?

This guide is designed to help Database Administrators and Key Clients use Key Connection to implement key retrieval and encryption for Microsoft’s SQL Server. It begins with application and security concepts, then continues with installation, configuration, and chapters on using SQL Server TDE with Key Connection and using SQL Server cell level encryption with Key Connection. It ends with chapters on performance tuning, backup and recovery, technical support, error codes, and troubleshooting.

## Previous versions of SQL Server

Microsoft SQL Server 2008 was the first version of SQL Server that supported the EKM function. If you need support for previous versions of SQL Server, please contact Townsend Security for further support.

## Other resources

The following documents provide additional information on the installation and use of Alliance Key Manager:

* [AKM User Guide]({{site.baseurl}}/akm_user_guide/)

* [AKM Server Management Guide]({{site.baseurl}}/akm_server_management_guide/)

* [AKM Administrative Console Guide]({{site.baseurl}}/akm_administrative_console_guide/)

## Notices

This product and documentation is covered by U.S. and International copyright law. This product may incorporate software licensed under one or more open source license agreements. Government users please note that this product is provided under restricted government use license controls. Please refer to the [AKM End User License Agreement]({{site.baseurl}}/eula/) for more information.

## Change Log

The following table provides information on the changes to this documentation:

**Version** | **Date** | **Description**
--- | --- | ---
**1.0.0** | 5/7/2011 | Initial release.
**1.0.1** | 5/21/2011 | Added additional installation and configuration information. Added SQL Server examples.
**1.02** | 9/1/2011 | Restructure the sections on TDE and cell level encryption. Some minor corrections.
**1.03** | 12/7/2011 | Added new documentation and notes to clarify many operations.
**3.0.0.001** | 1/14/2014 | New manual format and updates.
**3.0.0.002** | 10/15/2014 | Update Before You Begin chapter. 
**3.0.3.001** | 12/16/2014 | Update for AKM 3.0.3 and the ready to use version of AKM for VMware. 
**3.0.3.002** | 10/5/2015| Remove Key Connection licensing information. Update section on key rollover. Add section on transitioning from native TDE to using an external key manager. Misc doc updates.
**4.0.0.001** | 2/17/2016 | Update for AKM 4.0 and version 2.0 of Key Connection. Update pre-requisites.
**4.0.0.002** | 5/24/2016 | Update installation information, prerequisites, and preparation chapter for AKM 4.0 HSM release. Add section on limitations for cell level encryption.
**4.0.0.003** |  7/12/2016 | Update for AKM 4.0 Azure release. 

# Chapter 2: Application Concepts

## Extensible Key Management

Extensible Key Management (EKM) is the SQL Server facility provided by Microsoft to enable the use of external key management systems. When you enable and configure EKM, your encryption keys are stored on the AKM server and you may optionally perform remote encryption tasks on the server. 

## SQL Server instances

SQL Server supports installation of multiple instances on one server. Each instance runs as its own service on the Windows Server. Key Connection supports the use of multiple SQL Server instances. You can assign different security policies to different instances, and different instances can use different AKM servers. This provides full support for separation of duties and role-based access to encryption keys and services.

## Key Connection for SQL Server

Townsend Security’s Key Connection for SQL Server (Key Connection) provides a complete database encryption solution for Microsoft SQL Server 2008 and later when used with Alliance Key Manager. Key Connection enables SQL Server to communicate with AKM. On the Windows server, Key Connection receives key retrieval and encryption requests from SQL Server and communicates with the AKM server, which creates and stores encryption keys and performs encryption services. With Key Connection, users can easily deploy SQL Server’s Transparent Data Encryption (TDE) or Cell (Column) Level Encryption.

## Key Connection for SQL Server architecture

Key Connection is comprised of two parts: 

1. An EKM Provider module that is registered to EKM
2. A Windows service that performs key registration, key retrieval, encryption services, and communications with the AKM server

The EKM Provider module communicates with the Key Connection managed code using a secure Distributed Component Object Model (DCOM) interface. This provides a resilient and secure implementation based on Microsoft best practices for development. All Key Connection components use native Windows security and do not bypass any normal Windows security mechanism.

## Limitations

Key Connection does not support Fibers, also known as lightweight threads. Microsoft no longer recommends the use of Fibers with SQL Server 2008 to improve performance, and the Key Connection does not support any implementation that uses Fibers.

Key Connection does not support creating or deleting keys on AKM. This is a function reserved for a Crypto Officer using the AKM Administrative Console. The Crypto Officer must use the Admin Console to create, revoke, or delete encryption keys. This limitation is necessary to implement encryption key best practices and to meet regulations such as PCI Data Security Standard (PCI DSS). In SQL Server with EKM, you can use the following command:

```
CREATION_DISPOSITION = OPEN_EXISTING
```

You cannot use the following command:

```
CREATION_DISPOSITION = CREATE_NEW
```


Dropping a key from SQL Server using the `REMOVE PROVIDER KEY` is not supported. The key will be dropped from SQL Server, but not from the AKM server.

## Transparent Data Encryption (TDE)

Transparent Data Encryption (TDE) is a method of whole database encryption that can be enabled in SQL Server. When enabled, TDE encrypts the entire SQL Server database. When you use TDE in conjunction with Key Connection, the TDE encryption keys are protected by AKM. All encryption functions are performed by SQL Server once the TDE encryption key has been decrypted by AKM. Deploying TDE for SQL Server does not require any application code changes or changes to your existing SQL applications.

### Key protection

When you use TDE as your database encryption approach, the asymmetric keys used to protect TDE never leave the AKM server. When SQL Server starts, it will ask Key Connection to decrypt the key used for TDE to make it available, and then SQL Server will protect that key while SQL Server is active. The following graphic illustrates this architecture:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_2.jpg)

### Transitioning from using native TDE to using an external key manager

When using native TDE, you encrypt data with a symmetric key and then encrypt the symmetric key with a certificate or asymmetric key stored in master. However, you can instead use an asymmetric key stored on the AKM server to encrypt symmetric keys for additional security. 

After you install and configure Key Connection, follow the steps in the section [Create new KEK](#create_new_kek) to create an asymmetric key on AKM to use as the KEK, create a key alias for the new KEK in SQL server, and re-encrypt the DEK with the new key.

## Cell level encryption

Microsoft SQL Server provides a mechanism for encrypting a column in a database table called cell level encryption. When you use cell level encryption with EKM, the entire database is not encrypted, only the specified column. To use cell level encryption you must modify your SQL statements that read, insert, or update the column to encrypt or decrypt the data. Cell level encryption may provide better performance in certain implementations. Please refer to the Microsoft SQL Server documentation for a discussion on performance with EKM.

Cell level encryption supports the EKM architecture and you can store the encryption keys used for cell level encryption on the AKM server. SQL Server retrieves the symmetric key from AKM and then performs cipher operations. 

When you encrypt with an AKM symmetric key, Key Connection always uses the current key. When a key is rotated (rolled) manually or automatically, the new version (instance) of the key is used within 24 hours of its creation. For columns that were encrypted with an older version of the key, Key Connection uses the original version of the key for decryption.

### Key caching

If you use cell level encryption, Key Connection will securely cache the symmetric encryption key on the Windows server using the native Microsoft DAPI interface for key storage. When the Key Connection Windows service is ended the cached keys are erased and all local memory is cleared. Encryption keys are never transmitted over the network in the clear. 

If the network connection between the Windows Server and the AKM server fails, the encryption keys remain cached on the Windows server. If Key Connection cannot communicate with the key server, error events will be written to the Windows Event Manager, but will still continue to operate smoothly with SQL Server. When the network connection is restored, Key Connection will refresh the information about the encryption key the next time this is needed. This ensures that changes to encryption key access policies made by a Crypto Officer are reflected in the SQL Server 2008 environment within a 24-hour time period. 

When you change the Key Connection configuration, encryption keys are automatically refreshed from the AKM server and any changes to the key policies are also refreshed.

# Chapter 3: Security Concepts

## Dual control

Dual control is an important concept in most security regulations. For example, the PCI Data Security Standards (PCI DSS 3.0) specifically affirms the need for dual control on all sensitive key management activities. Alliance Key Manager supports a dual control option for key management tasks. When enabled in the AKM configuration file, you can only perform key management tasks when two different Crypto Officers authenticate to the key manager. See the [AKM Server Management Guide]({{site.baseurl}}/akm_server_management_guide/) for information on modifying the AKM configuration file.

## Separation of duties

Separation of duties is also an important concept for PCI and other compliance regulations. Separation of duties in a key management context means that the individuals who maintain encryption keys should not be the same as those individuals who have access to the data they protect. In practical terms this means that the SQL Server database administrator should not be the same person as the Alliance Key Manager Crypto Officer. By segmenting these responsibilities to different people in your organization, you reduce the chance of data breaches.

Townsend Security’s Key Connection fully supports separation of duties by implementing all key management functions on the AKM server and all database administration and EKM configuration tasks on the Windows Server. 

## Key rollover

Good security practice and many compliance regulations require that you roll (change) your encryption keys on a periodic basis. Asymmetric keys used for TDE are not automatically rolled, as SQL Server provides no mechanism to automatically roll TDE keys. 

> ***NOTE:***  During a roll operation, database maintenance activities will be briefly suspended. However, this will not cause any downtime of the application database, which will still be accessible by end users. 

<p class="anchor"><a name="create_new_kek" >&nbsp;</a></p>

### Create new KEK

You can effectively roll asymmetric keys by creating a new key on AKM, then re-encrypting the DEK with the new key. For this option, you do not have to turn off TDE, and you do not have to rotate the DEK. 

First create a new asymmetric key pair within the AKM Administrative Console using the "Create EKM Key" and the “Enable Key for EKM” commands. 

Then return to SQL Server and call the following command to create the asymmetric key alias for the new KEK that you created on the AKM server:

```
use master;
create asymmetric key my_new_kek from provider KeyConnection with provider_key_name = ’NEW_TDE_KEK’, creation_disposition = open_existing;
```

In this example, `NEW_TDE_KEK` is the name of the new key on AKM, and `my_new_kek` is the key alias.

Then use the `ALTER DATABASE` statement to re-encrypt the DEK with the new KEK alias assigned in the previous statement:

```
ALTER DATABASE ENCRYPTION KEY
ENCRYPTION BY SERVER 
     
        ASYMMETRIC KEY my_new_kek
```

### Roll the DEK

Another option is to alter the database encryption key with the following syntax: 

```
ALTER DATABASE ENCRYPTION KEY
      REGENERATE WITH ALGORITHM = AES_256
```

In SQL Server this is called “DEK rotation”. Note that SQL Server policies require you to back up your database logs after you roll the DEK twice, and attempting to roll the DEK a third time without backing up the logs will produce an error.
    
### Roll the DEK and create new KEK

A third option is to rotate the DEK and swap in a new AKM key at the same time:

```
ALTER DATABASE ENCRYPTION KEY
   REGENERATE WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER
   ASYMMETRIC KEY yet_another_kek
```

## System logs

Good security practice includes the collection and monitoring of system logs. Key Connection fully supports this goal in the Key Connection software and in the AKM server. The Key Connection software collects application activity in the Windows Event Manager where it can be collected and forwarded to a log collection server or Security Information and Event Manager (SIEM) application. AKM records all key management and key retrieval activity on the server and forwards this information in real time using the syslog facility on the server. You can fully comply with security best practices and compliance regulations through these facilities.

## SQL Server and EKM Auditing

Windows Server and SQL Server support additional auditing capabilities. Please refer to the Microsoft Windows and SQL Server documentation for information on how to enable the creation of SQL Server audit logs. See the following link for more information:
	
* https://msdn.microsoft.com/en-us/library/cc280386.aspx

## Windows and SQL Server Security

There are many other areas of good security practice that are not covered in this guide. You should refer to the Microsoft Windows and SQL Server documentation on steps you should take to secure your server and database environments. This includes the creation of user access policies for tables and columns. See the following link for more information about Microsoft SQL Server security best practices:
	
* https://msdn.microsoft.com/en-us/library/ms144228.aspx

# Chapter 4: Preparation

## Overview

Before setting up key retrieval in your client application, you will need to complete the following steps:
 
* Review prerequisites

* Determine encryption needs

* Install and set up the primary AKM server and any secondary mirror servers (instructions are located in platform specific deployment guides) 

* Create encryption keys (available as an option during AKM server setup, or, through the AKM Administrative Console application)

* Download certificates from the AKM server

* Know the IP address(es) of the AKM server(s) and port numbers for the desired services (key retrieval or remote encryption) 

See below for more information.
 
## Pre-requisites


### AKM version

Key Connection requires AKM version 4.0 or later. This version of Key Connection only supports TLS version 1.2.

### .NET Framework

Key Connection requires Microsoft .NET Framework 4.5.

### SQL Server version

Microsoft SQL Server Enterprise versions 2008+ support EKM.  

### Windows version

Key Connection is supported on the following Windows platforms:
 
* Windows Server 2012 
* Windows Server 2012 R2
* Windows Server 2008 (32-bit, 64-bit)
* Windows Server 2008 R2 (32-bit, 64-bit)

## Determine encryption needs

You will need to determine whether you wish to use Transparent Data Encryption (TDE) or cell level encryption. If you will use TDE, see [Chapter 5: Requirements for TDE](#chapter_5). If you will use cell level encryption, see [Chapter 6: Requirements for Cell Level Encryption](#chapter_6).
 
## AKM licensing

The AKM server supports managing keys for multiple Windows Servers and SQL Server instances. A temporary or permanent license is required to use or evaluate AKM. All deployments of AKM create a 30-day license automatically during setup and initialization, except for the Amazon Web Services fee-based deployment, which generates a permanent license.

A temporary license will enable a fully functional AKM server that may be run in your environment for evaluation or testing. If the temporary license expires, a permanent license may be purchased from Townsend Security or your software vendor. See your AKM platform specific deployment guide for information on installing a permanent license.
 
## Certificates and private keys

The client and AKM server use certificate and private keys to establish a secure TLS connection and perform authentication. You will need to install the following certificates and private keys on the client in order to authenticate your client application with the AKM server:
 
* AKM’s certificate authority (CA) certificate in `.pem` or `.der` format. This is the root CA certificate

* Client certificate/private key. These must be installed as one file in PKCS #12 format (`.p12` or `.pfx`)

These certificates are generated on initialization and stored on the AKM server. See your platform specific AKM deployment guide for instructions on downloading key client certificates.
  
> **_SECURITY ALERT:_** Private key files must be protected during creation, distribution, and storage to prevent loss. The loss of these files will compromise the security of the AKM server. Depending on the file format, the private key files may be bundled with a certificate or they may be separate files. Transfer the private key files by sharing them over a secure network, placing them in a password-protected zip file, sending them using SFTP, or another secure method. Use the same level of care you would employ to protect encryption keys, including encryption. In the event the private keys are compromised or lost, you should immediately replace the certificate authority on the AKM server and all client certificates in that chain of trust. See the [AKM Certificate Manager Guide]({{site.baseurl}}/akm_certificate manager_guide/) for more information.

 
## Server information

The following server information is required to set up Key Connection:

* The IP address or DNS name of the primary AKM server and any secondary AKM servers

* The port number for key retrieval services on AKM (the default is 6000) or remote encryption (the default is 6003)

## Encryption keys

To set up key retrieval or remote encryption, you must have the name(s) of the encryption key(s) on AKM you would like to use. When using TDE, you will need an asymmetric key pair. When using Cell Level Encryption, you will need a symmetric key. See later chapters for more details. 

AKM setup and initialization includes the option to generate an initial set of encryption keys. See your platform-specific AKM deployment guide for more information on encryption keys available for use, if that option was selected.

If needed, encryption keys can be created using the AKM Administrative Console application. See later chapters for more details. 
 
## Checklist

Before continuing, you will need the following items:
 
* AKM’s CA certificate in `.pem` or `.der` format

* A client certificate/private key in in PKCS #12 format (`.p12` or `.pfx`)

* The IP address or DNS name of the primary AKM server (and any secondary AKM servers) and the key retrieval services port number (the default is 6000) or remote encryption services port number (the default is 6003)

* The name of one or more encryption keys on the AKM server

<p class="anchor"><a name="chapter_5" >&nbsp;</a></p>

# Chapter 5: Requirements for TDE

If you are configuring and using Transparent Data Encryption (TDE), you will need to complete the following steps with the help of your System Administrator and Crypto Officer:

* Receive server information 
* Enable encryption for TDE
* Create asymmetric keys for TDE
* Enable asymmetric keys for TDE
* Enable key mirroring

See the sections below for more information. 


> **_IMPORTANT:_**  If you are configuring and using cell level encryption, see [Chapter 6](#chapter_6) of this guide.

## Receive server information 

You will need to receive the following information from your System Administrator:

* host name or IP address of the AKM server
* key retrieval and encryption port information, if changed from the default

## AKM configuration file

Encryption must be enabled in the AKM configuration (`akm.conf`) files of both the primary and secondary AKM servers, so that Key Connection will be able to connect to the "encryption port" when encrypting and decrypting with an asymmetric key pair (the "EKM Key"). Encryption is enabled by default. 

See the [AKM Server Management Guide]({{site.baseurl}}/akm_server_management_guide/) for more information on encryption options in the AKM configuration file.

## Creating asymmetric keys for TDE

If you are using the TDE facility of SQL Server, you must create an asymmetric key pair. During initialization of the AKM server, you had the option to generate an initial set of encryption keys, including an asymmetric key pair enabled for TDE. See your deployment guide for more information on keys available for use. 

If you need to create an asymmetric key pair, you can do so using the AKM Administrative Console. This is usually performed by a Crypto Officer. See below for more information.

Open the AKM Administrative Console. Select **_Create EKM Key_** under **_Key Connection for SQL Server_**. The following panel is displayed:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_3.jpg)

Enter a user-friendly name for the asymmetric key pair and select a key size. You can choose 1024 or 2048 bit for the key size. It is recommended that you select a 2048 bit key. Click **_Submit_**.

## Enabling the asymmetric key for TDE

After creating the asymmetric key for Microsoft EKM TDE encryption, you must enable it for use by EKM. This gives the Crypto Officer full control over the encryption key and its use in the SQL server environment. 

> **_NOTE:_**  Encryption keys never leave the AKM server, and are never displayed in the clear.

In the Admin Console, select the option to **_Enable Key for EKM_** under **_Key Connection for SQL Server_**:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_4.jpg)

Enter the name of the key you just created and click **_Submit_**. You can disable the key for use by EKM at any time in the future without deleting or revoking the key using the **_Remove Key for EKM_** command.

You can now share the name of this key with your SQL Server Database Administrator, who can then create data encryption keys for use with SQL Server and enable TDE. See [Chapter 9: Use TDE with Key Connection](#chapter_9) for instructions.

## Note on key mirroring

Alliance Key Manager supports real time mirroring of encryption keys to secondary AKM servers. This provides support for high availability and disaster recovery implementations. EKM keys are automatically mirrored to any secondary servers which have been set up to receive mirrored keys. See the [AKM User Guide]({{site.baseurl}}/akm_user_guide/) for more information on mirroring.

<p class="anchor"><a name="chapter_6" >&nbsp;</a></p>

# Chapter 6: Requirements for Cell Level Encryption

If you are configuring and using cell level encryption, you will need to complete the following steps with the help of your System Administrator and Crypto Officer:

* Receive server information
* Create symmetric keys for cell level encryption 
* Enable symmetric keys for EKM

See the sections below for more information.


> **_IMPORTANT:_**  If you are configuring and using Transparent Data Encryption (TDE), see [Chapter 5](#chapter_5) of this guide.


## Server information

You will need the following information from your System Administrator:

* the host name or IP address of the AKM server
* key retrieval and encryption port information, if changed from the default


## Create symmetric keys for cell level encryption

If you are using the cell level encryption facility of SQL Server you must have a symmetric key available on the AKM server. During initialization of the AKM server, you had the option to create an initial set of encryption keys, including symmetric encryption keys enabled for EKM. See your deployment guide for more information on keys available for use. 

If you need to create symmetric encryption keys, you can do so using the AKM Administrative Console. This is usually performed by a Crypto Officer. See below for more information.

Start the AKM Administrative Console and use the following steps to create a symmetric key. 

Under **_Manage Keys_**, select the option to **_Create Symmetric Key_**:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_5.jpg)

**Key Name:** A symmetric key should have a user-friendly name for use with EKM. 

**Key Size:** You can select 128-bit, 192-bit, or 256-bit key size for use with EKM.

**Activation Date:** Select an activation date for the key, or check the box to **_Activate key immediately_**. Note that the key will not be available before the activation date.

**Expiration date:** Select an expiration date for the key, or check the box for **_Key never expires_**. Note that the key will not be available after the expiration date. If you choose the option to never expire the key, the key will be available until it is deleted from the server.

**Rollover Code:** Select an option for how you would like the key to be rolled and a new instance of the key generated. If you choose **_Automatic_**, new key instances will be automatically created based on the value entered in the Rollover Days field. If **_Manual_**, new key instances will need to be created manually via calls to the “Rollover” command. If **_Never_**, new key instances will never be created. The Rollover Code can be changed at a future time.

> **_SECURITY ALERT:_** Good security practice and many compliance regulations require that you roll (change) your encryption keys on a periodic basis. EKM provides a mechanism for rolling symmetric encryption keys used for cell level encryption. Key Connection supports this key rollover architecture.  

> Alliance Key Manager provides support for both automatic key rollover and manual key rollover of symmetric keys. The Crypto Officer establishes the key rollover policy at the time the symmetric key is created, but can change it at any time. Key Connection will detect the presence of a newly rolled key and use it for new encryption tasks.

**Deletable:** This option determines whether it will be possible to delete the key. It is recommended that you select **_No_** to make the key non-deletable. This is because if you delete the key, it cannot be re-created, and you will need to restore the key manager from a backup to recover the key.

**Mirror Key:** Select the key mirror option you want for this key. If you have a secondary AKM server set up to receive mirrored keys, you can select **_Yes_** for the mirror attribute if you wish to mirror this key. If you are not using key mirroring, select **_No_** for this option. 

**Key Access:** Set the key access option you want to enforce for this key. You can restrict the key to a user and/or group. 

> **_IMPORTANT:_** Microsoft EKM does not provide user information to the key manager. However, you can restrict access to keys across multiple instances of SQL Server by assigning user and group information to the key. The User Access and Group Access fields must match the CN (Common Name) and the OU (Organizational Unit) on the client certificate, respectively. See the [AKM User Guide]({{site.baseurl}}/akm_user_guide/) for more information on certificates.

## Enable symmetric keys for EKM

After you create symmetric or asymmetric encryption keys, you must enable them for use with EKM. Use the Admin Console to enable encryption keys for use by EKM. Under the section **_Key Connection for SQL Server_**, select the option to **_Enable Key for EKM_**:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_6.jpg)

Enter the symmetric key name and click **_Submit_** to enable the key for use by EKM. You can disable the use of the key at any time in the future without revoking or deleting the key.



# Chapter 7: Install Key Connection



The Key Connection install file is located in the following directory on the AKM Supplemental:

```
AKM_Supplemental\Client_Platforms\Windows\Key_Connection\Install
```


Double-click `KeyConnection_[version]_[platform].msi` to begin the installation.

The following panel is displayed:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_7.jpg)

Click **_Next_** to start the installation of the Key Connection software.

The following panel is displayed:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_8.jpg)

Read the license agreement and check the option to accept the license terms. If you do not accept the license terms, click **_Cancel_** to stop the installation. Click **_Next_** to continue the installation.

The following panel is displayed:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_9.jpg)


If you accept the default installation location, click **_Next_** to continue. Otherwise, enter a path for the installation and click **_Next_** to continue.

The following panel is displayed:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_10.jpg)

Click **_Install_** to install the Key Connection software.

The following panel is displayed:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_11.jpg)

Click **_Finish_** to complete the installation. You can now configure Key Connection.



# Chapter 8: Configure Key Connection

After you install Alliance Key Manager and Key Connection, you must configure both for use with EKM. This is done using the Configuration Tool included with Key Connection. You can run the configuration application any time after installation.

Launch the Key Connection Configuration Tool from your Start menu, desktop, or Start screen. The following panel is displayed to let you configure the primary AKM server IP address and port number:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_16.jpg)

Enter the host name or IP address of the AKM server. You should have already received this information from your System Administrator. 

If you want the local system to accept a key server certificate that does not contain the host name, check the box to **_Accept the key server certificate_**. This does not disable certificate security authentication, and will avoid extraneous error messages during connections by Key Connection to the key server.

Enter the port numbers for the key retrieval and encryption ports. The default values (6000 and 6003 respectively) are the normal ports used by the key server. Your System Administrator will give you different port numbers if they have been changed.


> **_IMPORTANT:_**  Modify your local and external firewall rule so that ports 6000 through 6003 are open.

Click **_Next_** to continue.


> **_A NOTE ON TERMINOLOGY:_**  A certificate authority certificate is usually called a “CA certificate”, and an Intermediate certificate authority certificate is called an “Intermediate CA certificate”. In Microsoft documentation the equivalent term most often used is “Authoritative certificate”. This type of certificate should be distinguished from a Client certificate.

On the next panel you will define the Root CA certificate or the Intermediate CA certificate of the issuing Certificate Authority. The client certificate issued by this Certificate Authority will use this CA Certificate to authenticate the AKM server. 

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_17.jpg)

The drop-down box shows all certificates in both Trusted Root CA and Intermediate CA (the full names are “Trusted Root Certification Authorities” and “Intermediate Certificate Authorities”) local computer (certificate) stores for Local Computer. 

You can use the **_Import_** button to import a certificate file from your hard drive. The certificate will be added to the Windows certificate store. If you import a certificate that already exists, the drop-down box will tell you it was already there and will not overwrite the existing certificate (since that could remove properties you had set on it).

This panel shows the drop down list of certificates and thumbprints for the certificate store:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_18.jpg)

Select the Root CA certificate or intermediate CA certificate you want to use.

You can filter the certificate names by typing a partial name in the Search field. This includes searching on the hex values in a thumbprint:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_19.jpg)

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_20.jpg)


After you define the CA certificate for the connection to the AKM server, you will see the summary configuration panel. You can add additional AKM servers or continue with configuration:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_21.jpg)

Note that once you have configured certificates, the Configuration Tool always opens to this page (the list of key servers configured).

 Click **_Next_** to continue.
 
You must now select the client certificate for the connection to the key server:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_22.jpg)

This time, the drop-down box shows certificates in the Personal store of the local Computer. The client certificate may already be in your Windows certificate store, or more likely you will need to import an AKM-generated client certificate. Select the P12 type client certificate.

Note that the SQL Server instance name shows in the title bar. For example, it might show `MSSQLSERVER` which is the default instance name. If you have more than one instance installed, you will see a “List of clients” page and you would select `MSSQLSERVER`, for example, and then click **_Modify_** to get the correct instance of SQL Server.

Once you have imported and selected the client certificate, this panel will show the certificate CN and OU as informational fields to remind you of the key access that this certificate gives you on the key server:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_23.jpg)

Click **_Next_** to continue. The Configuration Tool detects whether or not SQL Server has read access to the private key of this certificate (which it needs to make the SSL connection to the key server) and prompts if it needs to grant it.

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_24.jpg)

Click **_Yes_**.
On the next panel you will select the key server to be used with this instance of SQL Server. You will see a list of key servers that you defined earlier. Select the server from the drop down list. This is also where you would find a failover (secondary) key server if you defined one previously:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_25.jpg)

Click **_Save_** to continue.

The first time you save the SQL Server client configuration, the Configuration Tool will detect whether or not SQL Server has DCOM access to the Key Connection service running on the local Windows platform (this Windows service was installed by the Key Connection installer). If it does not, the tool will prompt you to grant access:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_26.jpg)

Click **_Yes_**. This is the equivalent of granting access using the standard Windows DCOMCNFG tool.

Upon completion all of the SQL Server instances will be listed here. The Key server and Client certificate thumbprint fields will be blank if the instance is not yet configured. The Clear button will clear these fields for the selected instance thereby removing the configuration.

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_27.jpg)

Click **_Finish_**.

The SQL Server Test button will run a simple test to see if the tool can connect to SQL Server and list keys on the AKM Server using the EKM provider. The test runs to SQL Server, and does not bypass normal security or access controls. The following panels show the results of running the SQL Server test.

If the provider has not yet been created on SQL Server, the tool will prompt you, asking if it should be created now:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_28.jpg)

Click **_Yes_** to create the cryptographic provider for Key Connection. You must have SQL Server administrative rights for this option to succeed. The Configuration Tool will not escalate your privilege to grant you access if you do not already have it. If you are unable to create the cryptographic provider object in this step, see the section Create symmetric key alias in Chapter 10 for the SQL commands to use.

When you ask the Configuration Tool to create the cryptographic provider, you will be prompted to pick the name for the provider. You can choose any name you like, but it must be used in your SQL statements the way it is entered here. The name “KeyConnection” is used by default:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_29.jpg)

Click **_OK_** to continue.

If SQL Server has not yet been enabled for EKM, the tool will detect this and ask you if it should enable EKM:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_30.jpg)

Click **_Yes_**.
The following is an example of the resulting display of the SQL Server test:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_31.jpg)

The test collects any event log messages sent by the SQL Server instance and/or Key Connection and will display them in this window so that you do not need to switch to the event log viewer if there is a problem.

If there are errors in the SQL Server test to the key manager, you can view the events in the Windows event log:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_32.jpg)

Here is an example of an event log message. In this case we tried to create a key from SQL Server (from a SQL command prompt in SQL Server Management Studio) for a non-existent key.

Note that the key server error 4444 is noted in the message text. This will match the error number in the AKM server error log (`akmerror.log`). Any Key Connection error in the range 3000 to 4999 will match the same error code used in the AKM Server. Any Key Connection error under 3000 is local to the Windows client.

For information on how to view event logs in the Windows Event Viewer, see [Chapter 15: Troubleshooting](#chapter_15). For more information on Error Codes, See [Chapter 14: Error Codes](#chapter_14).

## Next steps
If you are using SQL Server TDE with Key Connection, continue to Chapter 9. If you are using cell level encryption, see [Chapter 10](#chapter_10).

<p class="anchor"><a name="chapter_9" >&nbsp;</a></p>


# Chapter 9: Use TDE with Key Connection

After you have configured Key Connection, you can begin using SQL Server TDE with Key Connection.

> **_NOTE:_** This section describes defining and enabling SQL Server Transparent Data Encryption (TDE). If you want to use SQL Server cell level encryption, see [Chapter 10](#chapter_10) of this guide. 


## Enable TDE

To enable TDE, use the following steps:

1. Create an asymmetric key and enable it for EKM (See [Chapter 5](#chapter_5))
2. Create an asymmetric key alias
3. Create database encryption key and use TDE
4. Alter database to turn encryption on
5. Verify that the database is encrypted

See the sections below for more information. 

### Create an asymmetric key alias

Use the following command to create an asymmetric key alias on SQL Server that refers to an existing enabled asymmetric key (RSA-KEY-1 in the example below) on the AKM Server. You must use the master database when creating the key:

> **_NOTE:_** be sure to keep a record of the alias you create (rsa_key_1 in the example below), as well as the enabled EKM key existing on AKM ('RSA-KEY-1' in the example below) that is used here. These will be needed should you ever need to restore a TDE encrypted database that was using an EKM from AKM. More details about SQL database restores can be found in [Chapter 12](#chapter_12) of this guide.


```
use master;
create asymmetric key rsa_key_1 from provider KeyConnection with provider_key_name = 'RSA-KEY-1', creation_disposition = open_existing;
```

### Create Database Encryption Key and use TDE

To use TDE, you need to create a special symmetric key called the “database encryption key”. You will create the database encryption key after switching to the database to be encrypted (“townsend” is an arbitrary database name used throughout this section as an example):

```
use townsend;
create database encryption key with algorithm = AES_256 encryption by server asymmetric key rsa_key_1;
```

SQL Server uses this symmetric key to encrypt and store your data. Although the key is stored by SQL Server, it is still protected. EKM protects it by encrypting the symmetric key itself. When SQL Server wants to work with encrypted data, it sends the encrypted symmetric key to AKM and requests that AKM use its asymmetric key (`rsa_key_1` in this example) to decrypt the symmetric key. AKM does this and sends the decrypted symmetric key back to SQL Server so that SQL Server can work with the data.

### Alter database to turn encryption on

There is one final step to enable TDE:

```
alter database townsend set encryption on;
```

### Verify that a database is encrypted

To check the encryption status of your database, view the following statement:

```
select*from sys.dm_database_encryption_keys
```

The column **database_id** displays the ID of the database. The column **encryption_state** indicates whether the database is encrypted or not encrypted and can contain the following values:

0 = No database encryption key present, no encryption
1 = Unencrypted
2 = Encryption in progress
3 = Encrypted
4 = Key change in progress
5 = Decryption in progress
6 = Protection change in progress (The certificate or asymmetric key that is encrypting the database encryption key is being changed.)

If the database has been encrypted, the value will be 3.

To view the names of the databases, view the following statement:

```
select*from sys.databases
```

The database names are displayed alongside the database IDs.


## Optional EKM-related SQL Server functions

The following additional features are supported by SQL Server: 

* Manually create cryptographic provider
* List EKM provider properties
* List enabled asymmetric keys available on the AKM server

See the sections below for further information on these optional SQL Server functions and commands.

### Manually create cryptographic provider

The EKM cryptographic provider is created by the Key Connection Configuration Tool and has the name `KeyConnection`. Using the Key Connection Configuration Tool is the preferred way to create the EKM provider. However, you can manually create the EKM cryptographic provider and give it another name you choose using the following command:

```
create cryptographic provider KeyConnection from file = 'C:\Program Files\Townsend Security\Key Connection for SQL Server\bin\EkmProvider.dll';
```

### List EKM provider properties

If you wish to list EKM provider properties of Key Connection, you can query the SQL Server management views using the following command (the `friendly_name` below is constant and should be used):

```
select provider_id from sys.dm_cryptographic_provider_properties where friendly_name = 'Key Connection for SQL Server';
```

You will need the provider ID returned by that last query for the next query shown below.

### List enabled asymmetric keys available on the AKM server

Keys must be both created and then enabled using the AKM Administrative Console (See [Chapter 5](#chapter_5)). After asymmetric keys have been created and enabled, you can list them from SQL Server using the following query:

```
select * from sys.dm_cryptographic_provider_keys(provider_id);
```

## Additional features

SQL Server also supports the following features: 

* Create a database encryption key for TDE that is protected by an AKM asymmetric key

Contact Townsend Security for more information. 

## More information

See the following websites for more information on TDE:

* https://msdn.microsoft.com/en-us/library/bb934049.aspx
* https://technet.microsoft.com/en-us/library/cc645957.aspx


<p class="anchor"><a name="chapter_10" >&nbsp;</a></p>

# Chapter 10: Using SQL Server Cell Level Encryption with Key Connection

After you have configured Key Connection, you can begin using SQL Server cell level encryption with Key Connection.

> **_NOTE:_** This section describes defining and enabling SQL Server cell level encryption. If you want to use SQL Server Transparent Data Encryption (TDE), see [Chapter 9](#chapter_9) of this guide.

## SQL Server supported features

The following features are supported by SQL Server:

* Create cryptographic provider
* List EKM provider properties
* List encryption keys available on the AKM server
* Create symmetric key alias
* Cell-level encryption with symmetric key
* Symmetric encryption with authenticators
* Cell-level size limits for symmetric encryption
* Determining the symmetric key name and instance id from the ciphertext

## Create cryptographic provider

The EKM cryptographic provider is created by the Key Connection Configuration Tool and has the name `KeyConnection`. Using the Key Connection Configuration Tool is the preferred way to create the EKM provider. However, you can manually create the EKM cryptographic provider and give it another name using the following command:

```
create cryptographic provider KeyConnection from file = 'C:\Program Files\Townsend Security\Key Connection for SQL Server\bin\EkmProvider.dll';
```

## List EKM provider properties

If you wish to list EKM provider properties, you can query the SQL Server management views like this:

```
select provider_id from sys.dm_cryptographic_provider_properties where friendly_name = 'Key Connection for SQL Server';
```

You will need the provider ID returned by that last query for the next query shown below.

The AES encryption uses CBC mode and the 128-bit initialization vectors are provided by SQL Server. 


## List encryption keys available on the AKM server

Encryption keys must be available on the AKM server. You can list available encryption keys from SQL Server using this query:

```
select * from sys.dm_cryptographic_provider_keys(provider_id);
```

If there are no encryption keys available, you can create encryption keys using the AKM Administrative Console.

## Create symmetric key alias

Here is a sample SQL command to “create” a symmetric key on SQL Server. The key actually refers to (is an alias of) an existing AKM Server key (that was previously created and enabled). The name used here, `my_key`, is how you refer to this key from SQL Server. The AKM Server key name seen here, `KEY01-128`, only appears in this `CREATE` syntax and then it is no longer needed for SQL Server commands using the key. The provider name seen here, `KeyConnection`, is the name if the EKM provider created by the Key Connection Configuration Tool:

```
create symmetric key my_key from provider KeyConnection with provider_key_name = 'KEY01-128', creation_disposition = open_existing;
```

You can see this new key listed in the following SQL Server management view:

```
select * from sys.symmetric_keys;
```

## Cell-level encryption with symmetric key examples

You can now encrypt and decrypt with this key using syntax like the following. This example just encrypts a string literal: for example, you can use a column name in place of `Hello World`. The important points are the SQL Server functions `encryptbykey`, `decryptbykey`, and `key_guid`. Also note that you do not need to use `key_guid` with `decryptbykey`. The third statement below is decrypting the immediate result of the encryption. You can copy and paste these example statements in the SQL console and the result will be `Hello World`.

This example illustrates a literal string:

```	
select encryptbykey(key_guid('my_key'), 'Hello World');
```

This example illustrates a column:

```
select decryptbykey(some_column) from some_table;
```

This example illustrates decrypting the result of the encryption:

```
select convert(varchar, decryptbykey(encryptbykey(key_guid('my_key'), 'Hello World')));
```

## Symmetric encryption with authenticators

You can also encrypt and decrypt with authenticators (see the following link for more information: https://msdn.microsoft.com/en-us/library/ms174361.aspx). For example:

```
create table my_table (encr varbinary(256));
insert into my_table values (encryptbykey(key_guid('my_key'), 'Hello World', 1, 'Townsend Security'));

select encr from my_table;
select decryptbykey(encr) from my_table;
select decryptbykey(encr, 1, 'Townsend Security') from my_table;
select convert(varchar, decryptbykey(encr, 1, 'Townsend Security')) from my_table;
```

The first query returns ciphertext in binary. The second returns `NULL` because the authenticator is missing. The next returns the plaintext as binary data, and the last returns the expected string `Hello World` (binary converted to varchar). 

## Cell-level size limits for symmetric encryption

The limit for cell-level encryption is 7,927 bytes of plaintext without authenticators, and 7,907 bytes with authenticators.

SQL Server adds 8 bytes of data to the plaintext which is passed to Key Connection, for an upper limit of 7,935 bytes.

This is rounded up to 7,936 bytes for OAEP padding, and then Key Connection adds an additional 16 bytes to the resulting ciphertext buffer to store the key instance for a total of 7,952 bytes returned to SQL Server.

SQL Server then adds an additional 36 bytes of data which is stored with the ciphertext, for a total of 7,988 bytes.

Adding just one more byte of plaintext to make it 7,928 bytes, with the extra 8 bytes that SQL Server puts in along with the plaintext, would result in 7,936 bytes of plaintext to encrypt. Since this is an even multiple of 16 (496 * 16), the OAEP padding would add 16 bytes of padding, for a total of 7,952 bytes of ciphertext. If we then add our key instance ID along with the ciphertext, we would return 7,968 bytes to SQL Server. Adding the 36 bytes of cell overhead, that would give a total of 8,004 bytes, which exceeds the SQL Server column maximum size of 8,000 bytes.

In order to support key rollover, we can reduce the maximum size of plaintext by 16 bytes, the size needed to store the key instance ID.


When authenticators are used, SQL Server adds an additional 20 bytes of overhead to the plaintext, reducing the total size of plaintext to 7,907 bytes.

**Symmetric Limits** | **SQL Server** | **Key Connection**
--- | --- | ---
**Without authenticators** | 7,943 | 7,927
**With authenticators** | 7,923 | 7,907

## Determining the symmetric key name and instance ID from the ciphertext

You can peek into the ciphertext result of a symmetric encryption and use the bytes that SQL Server stores in it, and also the 16-byte instance name that we store in it. The following are two samples of user-defined functions which interpret the ciphertext.

Using this function, you can use information stored by SQL Server to look up the name of the symmetric key used to encrypt the ciphertext:

```
CREATE FUNCTION keyname_from_ciphertext(@ciphertext VARBINARY(8000))
RETURNS NVARCHAR(128)
AS
BEGIN
  DECLARE @keyname NVARCHAR(128)
  SELECT @keyname = name FROM sys.symmetric_keys WHERE key_guid = convert(uniqueidentifier, substring(@ciphertext, 1, 16))
  RETURN @keyname
END
```

This function will give the Base64 instance name as displayed as `akmadmin --display-key-instance-list --key-name [name]`:

```
CREATE FUNCTION instance_from_ciphertext(@ciphertext VARBINARY(8000))
RETURNS VARCHAR(24)
AS
BEGIN
  DECLARE @instance VARBINARY(16)
  SELECT @instance = SUBSTRING(@ciphertext, 37, 16)
  RETURN CAST(N'' as XML).value('xs:base64Binary(xs:hexBinary(sql:variable("@instance")))', 'VARCHAR(24)')
END
```

For example, if you had a table `key64` with a column `ciphertext`, you can use these functions like this:

```
select distinct dbo.keyname_from_ciphertext(ciphertext), dbo.instance_from_ciphertext(ciphertext) from key64;
```


## Limitations For Cell Level Encryption

At this time SQL Server does not support impersonation for EKM Providers. Impersonation only works in a primary context. This means that only the user logged in on the SQL Server can encrypt and decrypt. Attempting to use `EXECUTE AS` in a statement to encrypt or decrypt a column will result in a NULL value.  

# Chapter 11: Performance Tuning

The Key Connection application runs as a Windows service and should have adequate resources to service your SQL Server encryption and key management needs. Be sure to provide adequate memory and resources to the Key Connection service for best performance.

When using cell level encryption, Key Connection uses the native Microsoft .NET encryption APIs for best performance. 


# Chapter 12: Backup and Recovery

## Backing up encryption keys

You should perform periodic backups of your key management database. In the event of the loss of a key management server due to fire or other natural catastrophe, or in the event of the failure of the key server hardware, you must restore the key management application and database from a backup. This should be performed by a System Administrator using the web interface. See the [AKM Server Management Guide]({{site.baseurl}}/akm_server_management_guide/) for information on how to backup and restore the encryption key database.

## Backing up Key Connection

You can use normal Windows Server backup utilities to save the Key Connection application. Unprotected encryption keys are not stored on the Windows server, and these keys will not be included in the backup.


> **_SECURITY ALERT:_**  No encryption keys or other sensitive cryptographic material are stored on these directories or on the Windows server. 

<!-- -->

> **_IMPORTANT:_**  Key Connection configuration information is stored in the Windows Registry. Please be sure that your backup operations include the Windows Registry.


## SQL Server mirroring

Some versions of SQL Server Enterprise support database  mirroring. Key Connection for SQL Server can be used in this environment as the key server can accept multiple simultaneous connections between multiple SQL Server instances. You must ensure that the certificates are properly configured in the Microsoft Windows certificate store before activating SQL Server mirroring. 


> **_IMPORTANT:_**  Each server that uses Key Connection must have the certificates needed to communicate with and authenticate the AKM server. Windows mirroring will not automatically mirror certificates.

## Restoring a TDE+EKM encrypted SQL database

To enable TDE, use the following steps:

1. Install Key Connection on the Windows Server that the SQL database will be restored to (See [Chapter 7](#chapter_7))
2. Configure Key Connection with the AKM's client certificates (See [Chapter 8](#chapter_8))
3. Recreate the asymmetric key alias in SQL
4. Restore the SQL database
5. Verify that the database is accessible 

See the sections below for more information. 

### Recreate the asymmetric key alias

When you recreate the key alias You must use the master database, and you must use the exact same alias (rsa_key_1 in the command below) that was specified on the database when TDE was originally implemented.  In addition to the alias you will need to specify the very same EKM key ('RSA-KEY-1' in the command below). that was originally used to encrypt the Database Encryption Key on SQL. The following command can be used to re-create the asymmetric key alias on SQL Server.

> **_IMPORTANT:_**  The variable 'RSA-KEY-1' refers to a key existing on AKM itself, you can check the list of available EKM keys in the AKM Administrative Console application.  

```
use master;
create asymmetric key rsa_key_1 from provider KeyConnection with provider_key_name = 'RSA-KEY-1', creation_disposition = open_existing;
```

At this point you can restore your SQL user database, and confirm that data is accessible.


# Chapter 13: Technical Support

Technical support is provided by your software vendor or by Townsend Security via the web at www.townsendsecurity.com.  You must have a valid, permanent AKM license to receive technical support. Please contact your account manager for more information.

<p class="anchor"><a name="chapter_14" >&nbsp;</a></p>

# Chapter 14: Error Codes

The following table provides the error codes for the Key Connection application. The values {0}, {1}, etc. are replaced with actual values in the Windows event manager log.

Error codes returned from the AKM server will be in the range of 3000 to 4999. You can find the description of these error codes in the Alliance Key Manager Error Codes Reference.

**Error code** | **Description**
--- | ---
2049 | ClientCertificateAuthenticationFailed. Client certificate has failed authentication connecting to key server {0}. {1}. {2}.
2050 | "ClientCertificateMisplaced. Local installed certificate matching configured client thumbprint ""{0}"" not found where it was expected. Instead  the certificate was found where the server certificate belongs. The subject name of this certificate is ""{1}"". If this is the client certificate then move it to the Personal certificate store."
2051 | ClientCertificateNoPrivateKeyAccess. The private key of the client certificate cannot be read by user '{0}'. Check the private key permissions. System message: {1}
2052 | "ClientCertificateNotFound. Local installed certificate not found matching configured client thumbprint ""{0}""."
2053 | ClientCertificatePrivateKeyMissing. Client certificate does not have a corresponding private key. Certificate thumbprint: {0}
2054 | ConfigClientNotConfigured. Key client {0} is not configured. Run the Configuration Tool to configure this client.
2055 | ConfigInvalidPortNumber. Invalid port number {0} configured key for server {1}.
2057 | ConfigThumbprintNotHexadecimal. Invalid value '{1}' for configured {0}. This value must be a hexadecimal string.
2058 | ConfigThumbprintOddCountHexadecimal. Invalid value '{1}' for configured {0}. This value must have an even number of hexadecimal digits.
2062 | CryptographicExceptionOnDecrypt. Cryptographic error occurred during decryption. Check the ciphertext length and padding.
2063 | DataStreamBadHeader. Unable to parse response header for key server request {0}.
2064 | DataStreamBadKeyDataLength. Bad key data length in key server response: expecting {0} bytes but received {1}.
2065 | DataStreamClosed. Key server connection was closed unexpectedly on server request {0}.
2066 | DataStreamParseError. Unable to parse incorrectly formatted key server response {0} at data offset {1}.
2067 | DataStreamReadCancelled. Key server connection read cancelled for request {0}.
2068 | DataStreamTruncated. Key server response for request {0} is truncated: expecting {1} bytes but received only {2}.
2069 | DataStreamTruncatedListItem. Key server response list item for request {0} is truncated: expecting {1} bytes but received only {2}.
2070 | DataStreamUnknownToken. Unknown token '{0}' in key server response {1}.
2071 | DataStreamWriteCancelled.Key server connection read cancelled for request {0}.
2072 | DataStreamWrongDataFormat. Wrong data format in key server response for request {0}: expecting {0} but received {1}.
2073 | DataStreamWrongResponseId. Wrong key server response {1} received while expecting {0}.
2074 | KeyAccessDenied. Access to provider key '{0}' instance '{1}' is denied to user '{4}' group '{5}'. Key thumbprint 0x{2}. Key server error {3}.
2075 | KeyExpired. Provider key '{0}' instance '{1}' has expired. Key thumbprint 0x{2}. Key server error {3}.
2076 | KeyInfoUnexpectedlyNotFoundById. Provider key information unexpectedly not found using key id {0}.
2077 | "KeyNotFound. Key name ""{0}"" not found on the key server."
2078 | KeyNotFoundByProviderName. Provider key '{0}' not found. Key server error {1}.
2079 | KeyNotFoundByThumbprint. Provider key not found using key thumbprint 0x{0} '{1}'. Key server error {2}.
2080 | KeyRevoked. Provider key '{0}' instance '{1}' is revoked. Key thumbprint 0x{2}. Key server error {3}.
2084 | NetworkConnectError. Network connect to key server at {0} port {1} failed. System message: {2}
2085 | NetworkHandshakeIOError. Key server connection handshake failed: {0}
2086 | NetworkReadIOError. Key server connection read failed for request {0}. {1}
2087 | NetworkWriteIOError. Key server connection write failed for request {0}. {1}
2088 | NotSupportedCreateKey. Create key with CREATION_DISPOSITION = CREATE_NEW is not supported. You must use CREATION_DISPOSITION = OPEN_EXISTING.
2089 | NotSupportedDropKey. Drop key with REMOVE PROVIDER KEY is not supported.
2090 | NotSupportedExportPrivateKey. Export of the private key of an asymmetric key pair is not supported.
2091 | NotSupportedExportSymmetricKey. Export of a symmetric key is not supported.
2092 | NotSupportedImportKey. Import Key is not supported.
2093 | RegistryAccessDenied. Access to Windows registry denied for key '{0}'.
2094 | RegistryEmptyValue. Required registry value '{1}' is found but empty for registry key '{0}'.
2095 | RegistryKeyNotFound. Required registry key is missing: '{0}'.
2096 | RegistryNotAuthorizedToValue .Insufficient access to Windows registry for key '{0}'.
2097 | RegistryValueNotFound. Required registry value '{1}' is missing for registry key '{0}'.
2098 | RegistryWrongValueKind. Wrong kind of registry value '{1}' which is '{2}' but must be '{3}' for registry key '{0}'.
2099 | "ServerCertificateMisplaced. Local installed certificate matching configured server thumbprint ""{0}"" not found where it was expected. Instead  the certificate was found where the client certificate belongs. The subject name of this certificate is ""{1}"". If this is the server certificate then move it to the Trusted Root or Intermediate Certificate Authorities "
2100 | ServerCertificateNameMismatch. Remote certificate name mismatch. Key server hostname '{0}' does not match the certificate's subject Common Name '{1}'. If you trust this key server  you can select the option in the Configuration Tool to accept the certificate without matching the hostname.
2101 | ServerCertificateNameMismatchWithAlt. Remote certificate name mismatch. Key server hostname '{0}' does not match the certificate's subject Common Name '{1}' or Subject Alternative Name '{2}'. If you trust this key server  you can select the option in the Configuration Tool to accept the certificate without matching the hostname.
2102 | "ServerCertificateNotFound. Local installed certificate not found matching configured server thumbprint ""{0}""."
2103 | "ServerCertificateNotInChain. The received key server certificate chain does not include a certificate having the configured server certificate thumbprint ""{0}""."
2104 | ServerCertificatePolicyError.Error in certificate chain received from key server '{0}': {1}. {2}. {3}.
2105 | ServerCertificatePublicKeyMismatch. Key server certificate public key does not match. Received public key: {0}. Local installed public key: {1}.
2106 | ServerDenyAdminOnKeyRetrievalPort. Administrator certificate is not allowed on key retrieval port. Request id {0}. Key server error {1}.
2107 | ServerFeatureNotSupported. Request {0} is not a supported feature for the installed version of the key server. Key server error {1}.
2108 | ServerShuttingDown. Key server is shutting down. Key server error {0}.

<p class="anchor"><a name="chapter_15" >&nbsp;</a></p>

# Chapter 15: Troubleshooting

## Windows Event Manager

When Key Connection encounters an error it writes detailed information to the Windows Event Manager. You can use standard Microsoft tools to view messages in the Event Manager. Key Connection will provide detailed information about the problem. If the problem originates on the AKM server there will be a key manager error code that you can use to look up the problem using the AKM Error Codes Reference. The following is an example of a Key Connection error logged to the Windows Event Manager:

![image alt text]({{site.baseurl}}/images/akm_key_connection_for_sql_server_user_guide/image_33.jpg)

## Key manager logs

The AKM server also collects detailed logs of any problems that are encountered. You can use the AKM Administrative Console to view messages created by the AKM key server. The logs are stored in the `/var/log/townsend` directory. The file `akmerror.log` contains summary information about key server activity. When verbose logging is enabled, the file `akmtrace.log` contains detailed information about any errors.


## Analyzing problems

You can use the Key Connection Configuration Tool to help analyze problems. Start the Configuration Tool and click **_Test Connection_** to view an interactive dialog of the connection to the key server. Any errors connecting to the server and listing keys will be displayed.






