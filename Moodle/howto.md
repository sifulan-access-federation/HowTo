# Moodle - SIFULAN Federation / eduGAIN Integration

## Objective

To integrate Moodle with SIFULAN Federation and eduGAIN

## Outcome

- Able to accept login from SIFULAN Federation / eduGAIN IdPs

## Introduction

This document provides a guide how to accept login from Identity Federation such as SIFULAN Federation and eduGAIN on Moodle. SIFULAN Federation and eduGAIN use SAML protocol to facilitate the Authentication (AuthN) process. Moodle has a built-in SAML plugin. However, it relies on another software/middleware called Shibboleth Service Provider (Shibboleth SP).  In general, there are 3 steps that we need to do:

1. Install and configure the Shibboleth SP
2. Configure Moodle’s SAML plugin
3. Register the Shibboleth SP metadata to SIFULAN Federation.

This guide assume that the Moodle is already installed on Apache Web Server on Linux and use the https protocol to access it.

## Install Shibboleth SP 

### For CentOS/Rocky Linux

1. Visit: ``https://shibboleth.net/downloads/service-provider/RPMS/`` to generate the yum repo file.
2. Select your OS distro and version, the click the generate button
3. Copy the output, then create the repo file and save it at the ``/etc/yum.repos.d/`` folder, for example:

   ```bash
   sudo vi /etc/yum.repos.d/shibboleth.repo
   ```

4. Update the repositories and install the Shibboleth SP:

   ```bash
   sudo yum update -y
   sudo yum install -y shibboleth
   ```

### For Ubuntu Linux

1. Install the Shibboleth SP:

   ```bash
   sudo apt install libapache2-mod-shib ntp --no-install-recommends
   ```

## Generate Metadata Signing and Encryption Keys

1. Change the current working folder to Shibboleth's configuration folder:

   ```bash
   cd /etc/shibboleth
   ```

2. Create SP metadata Signing and Encryption credentials:

   For CentOS/Rocky Linux:

   ```bash
   sudo ./keygen.sh -u shibd -g shibd -h <moodle’s hostname> -y 10 -e https://<moodle’s hostname>/shibboleth -n sp-signing -f
   sudo ./keygen.sh -u shibd -g shibd -h <moodle’s hostname> -y 10 -e https://<moodle’s hostname> /shibboleth -n sp-encrypt –f
    ```
   For Ubuntu Linux:

   ```bash
   sudo shib-keygen -u _shibd -g _shibd -h <moodle’s hostname> -y 10 -e https://<moodle’s hostname>/shibboleth -n sp-signing -f
    sudo shib-keygen -u _shibd -g _shibd -h <moodle’s hostname> -y 10 -e https://<moodle’s hostname> /shibboleth -n sp-encrypt –f
   ```

   Note:
   - Replace <moodle’s hostname> with your moodle’s hostname (e.g. moodle.university.edu.my)

## Configure Shibboleth SP

All Shibboleth SP configuration files are located at /etc/shibboleth folder. Some key configurations files are:

- shibboleth2.xml : the main configuration file
- attribute-map.xml : the attribute mapping configuration file

In addition, you also need to download SIFULAN Federation signer’s key to validate the metadata from SIFULAN Federation’s Metadata Query Service (MDQ) by using the following command:

```bash
sudo wget https://sifulan.my/metadata/sifulan-signer.pem -O /etc/shibboleth/sifulan-signer.pem
```

Should you change any configuration configuration file, you need to restart the shibd daemon by using the following command:

```bash
sudo systemctl restart shibd.service
```


