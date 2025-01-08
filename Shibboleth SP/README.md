# Shibboleth SP Installation Guide

## Preparation and Assumptions

- This installation guide assumes that you have a working apache website with SSL enabled.
- It's assumed that the server hostname is identical with the service hostname.

## Shibboleth Setup

### Ubuntu

- Install Shibboleth

  ```bash
  sudo apt install libapache2-mod-shib ntp --no-install-recommends
  ```

- Download IdP Metadata:

  ```bash
  sudo wget [IdP Metadata URL] -O /etc/shibboleth/idp-metadata.xml
  ```

  Replace ```[IdP Metadata URL]``` with the actual IdP Metadata URL (typically the entity ID name).

- Generate the keys for Shibboleth:

    ```bash
    sudo mkdir /etc/shibboleth/certs
    sudo shib-keygen -u _shibd -g _shibd -h https://[server hostname] -y 10 -e https://[server hostname]/shibboleth -n sp-signing -o /etc/shibboleth/certs
    sudo shib-keygen -u _shibd -g _shibd -h https://[server hostname] -y 10 -e https://[server hostname]/shibboleth -n sp-encrypt -o /etc/shibboleth/certs
    ```

  Replace ```[server hostname]``` with the actual server's hostname.

- Put aside the default ```shibboleth2.xml``` file:

  ```bash
  sudo mv /etc/shibboleth/shibboleth2.xml /etc/shibboleth/shibboleth2.xml.default
  ```

- Copy the content of [shibboleth2.xml](shibboleth2.xml) file from this repository, then paste it into ```/etc/shibboleth/shibboleth2.xml``` file:

  ```bash
  sudo vi /etc/shibboleth/shibboleth2.xml
  ```
- Change ```[server hostname]``` to server's hostname, ```[IDP's Entity ID]``` to organization's IdP Entity ID and ```[support email]``` to support's email inside the ```/etc/shibboleth/shibboleth2.xml``` file.

- Put aside the default ```attribute-map.xml``` file:

  ```bash
  sudo mv /etc/shibboleth/attribute-map.xml /etc/shibboleth/attribute-map.xml.default
  ```

- Copy the content of [attribute-map.xml](attribute-map.xml) file from this repository, then paste it into ```/etc/shibboleth/attribute-map.xml``` file:

  ```bash
  sudo vi /etc/shibboleth/attribute-map.xml
  ```

- Restart both Shibd and Apache:

  ```bash
  sudo systemctl restart shibd
  sudo systemctl restart apache2
  ```

## Apache Configuration

You MUST enable AuthType shibboleth for the module to process any requests, and there MUST be a require command as well. To enable Shibboleth but not specify any session/access requirements use "require shibboleth". For example:

```apache

<Location /secure>
  AuthType shibboleth
  ShibRequestSetting requireSession 1
  require shib-session
</Location>

```

So, when the user visit the URL ```https://[server hostname]/secure```, the user will be redirected to the IdP for authentication.

## Testing

- Open URL ```https://[server hostname]/Shibboleth.sso/Metadata``` in a web browser, you should see/download the Shibboleth metadata (file).
- The attribute(s)/claim(s) sent by the IdP is registered as environment variable(s) in the server. For example, you can use this variable ```$_SERVER['givenName']``` to get user's first name sent by the IdP in PHP Programming language.
