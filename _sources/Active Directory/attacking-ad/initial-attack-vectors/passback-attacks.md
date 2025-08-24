# Passback Attacks

### What is the intended functionality?

Network devices such as printers, scanners, or other appliances use LDAP authentication to validate users or access Active Directory services. These devices store credentials and automatically authenticate to the LDAP server when a user interacts with them, for example, to print documents or allow secure network access. The intended functionality is to simplify management and enforce centralized authentication. The LDAP settings are usually accessible via the device’s web-based configuration page, which can be reached by connecting to the device’s IP address in a browser. Access typically requires administrative credentials, which are often left at default values like admin:admin.

### What can go wrong here?

If an attacker gains access to the device’s configuration page, they can modify LDAP parameters, such as changing the server IP to a rogue LDAP server under their control. The device will then transmit its stored credentials to the attacker, who can capture them in plaintext if insecure authentication methods are used.

### Attack requirements

- Access to a network device’s configuration, typically via default credentials (admin:admin) or compromised accounts.

- Ability to modify LDAP parameters (e.g., IP/hostname of the LDAP server).

- Control over a rogue LDAP server that can accept insecure authentication methods (PLAIN/LOGIN).

- Internal network access so the device can reach the rogue server.

### Attack Walkthrough

1.  Gain access to the device configuration page

2.  Redirect the LDAP server to a rogue server

    - Set the LDAP server IP in the device configuration to your attack machine.

    - Start a simple listener (e.g., nc -lvp 389) to test connectivity.

    - Note: A plain Netcat listener may fail to capture credentials if the device negotiates a secure authentication method.

3.  Host and configure a rogue LDAP server

    - Install OpenLDAP and configure it to match the target domain.

      ```
      sudo apt-get install slapd ldap-utils
      ```

    - Downgrade authentication mechanisms to allow insecure methods (PLAIN/LOGIN) by creating an LDIF file (weak.ldif) and applying it:

      ```
      sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./weak.ldif
      sudo service slapd restart
      ```

4.  Capture credentials

    - The device now authenticates to your rogue LDAP server using the downgraded, insecure method.
    - Capture credentials in plaintext using tcpdump or another network sniffer:

    ```
    sudo tcpdump -SX -i <interface> tcp port 389
    ```

### Mitigations

- Change default passwords on all network devices and management interfaces.

- Enforce secure LDAP authentication (e.g., LDAPS, strong SASL mechanisms).

#### References

- [https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)
