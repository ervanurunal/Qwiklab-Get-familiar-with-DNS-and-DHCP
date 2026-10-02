## Qwiklabs: Get Familiar with DNS and DHCP

---

### Overview

In this lab, I worked with **dnsmasq** to manage DNS and DHCP services in a simulated network environment.

I modified an existing dnsmasq configuration to:

* Configure DNS and DHCP services
* Examine network interfaces
* Enable DNS query logging
* Test DNS queries
* Observe DNS caching behavior
* Test DHCP address assignment
* Configure fixed IP addresses for specific servers
* Modify the DHCP dynamic address range
* Change DHCP lease duration
* Verify configuration syntax
* Test the final DHCP configuration

The lab demonstrated how DNS and DHCP can be managed together in a smaller network environment using dnsmasq.

---

### Tools & Resources

* Linux Virtual Machine
* Linux Terminal
* `dnsmasq`
* `service`
* `ip`
* `ip link`
* `dig`
* `dhclient`
* `nano`
* `cat`
* `tail`
* `sudo`
* DNS
* DHCP

### Lab Type:
Linux / Networking / IT Support Hands-On Lab

---
---

## 1. Linux Commands Used

The lab introduced several Linux commands used throughout the configuration and troubleshooting process.

| Command    | Purpose                                        |
| ---------- | ---------------------------------------------- |
| `sudo`     | Execute commands with administrator privileges |
| `ls`       | List files and directories                     |
| `cat`      | Display file contents                          |
| `tail`     | Display the end of a file                      |
| `nano`     | Edit configuration files                       |
| `service`  | Manage services                                |
| `ip`       | View and configure network interfaces          |
| `dig`      | Perform DNS queries                            |
| `dhclient` | Request an IP address from a DHCP server       |
| `man`      | Display command documentation                  |

---

## 2. Lab Network Setup

The lab used a simulated network instead of making changes to a production network.

The DNS and DHCP server used the virtual interface:

```text
eth_srv
```

The client used:

```text
eth_cli
```

The server interface was configured with:

```text
192.168.1.1/24
```

The client interface initially did not have an IPv4 address.

---

## 3. Inspect Network Interfaces

The `ip` command was used to inspect the network configuration.

Example:

```bash
ip addr show eth_srv
```

The server interface was configured with:

```text
192.168.1.1/24
```

![1](https://i.imgur.com/2EGNw13.png)

The client interface could also be inspected:

```bash
ip addr show eth_cli
```

At the beginning of the lab, `eth_cli` did not have an IPv4 address.

![2](https://i.imgur.com/guuNc4W.png)

---

## 4. Examine the dnsmasq Configuration

The existing dnsmasq configuration was stored in:

```text
/etc/dnsmasq.d/mycompany.conf
```

I inspected the configuration using:

```bash
cat /etc/dnsmasq.d/mycompany.conf
```

![3](https://i.imgur.com/PgXB9Ur.png)

---

## 5. Understand dnsmasq Configuration

Important configuration options included:

#### `interface`

Specifies the network interface that dnsmasq uses to listen for DHCP requests and provide responses.

```text
interface=eth_srv
```

#### `bind-interfaces`

Restricts dnsmasq to the specified interface.

#### `domain`

Defines the domain name used by the network.

Example:

```text
domain=mycompany.local
```

#### `dhcp-option`

Provides additional network information to DHCP clients, such as:

* Default gateway
* DNS server

#### `dhcp-range`

Defines the range of IP addresses that can be dynamically assigned to clients.

The original configuration used a large portion of the available network for dynamic DHCP assignments.

---

## 6. Check the dnsmasq Service

The lab used the `service` command to check whether dnsmasq was running.

```bash
sudo service dnsmasq status
```

The service was running:

```text
dnsmasq(running)
```

![4](https://i.imgur.com/56zgHKj.png)

Checking service status is an important troubleshooting step before modifying configuration files.

---

## 7. Enable dnsmasq Debug Logging

Debug logging can help understand what a network service is doing.

First, stop dnsmasq:

```bash
sudo service dnsmasq stop
```

![5](https://i.imgur.com/AoiPmQG.png)

Then open the configuration file:

```bash
sudo nano /etc/dnsmasq.d/mycompany.conf
```

The lab added:

```text
log-queries
```

and configured a log file location.

![6](https://i.imgur.com/jCEYYxH.png)

After editing the file:

1. Press `Ctrl + X`
2. Press `Y`
3. Press `Enter`

---

## 8. Validate the dnsmasq Configuration

Before restarting the service, I checked the configuration syntax.

```bash
sudo dnsmasq --test
```

![7](https://i.imgur.com/BuKW89v.png)

A successful validation produced:

```text
dnsmasq: syntax check OK.
```

This step is important because configuration errors can prevent a service from starting correctly.

---

## 9. Start dnsmasq

After the configuration passed the syntax check:

```bash
sudo service dnsmasq start
```

![8](https://i.imgur.com/dB4FgUP.png)

The service could then be tested using DNS queries.

---

## 10. Test DNS with `dig`

The `dig` command can be used to request the IP address associated with a hostname.

Example:

```bash
dig @localhost example.com
```

![9](https://i.imgur.com/wR0rDVP.png)

In this lab, that means the query is sent to the running dnsmasq service.

---

## 11. Observe DNS Forwarding

After making the DNS query, I inspected the dnsmasq debug log.

![10](https://i.imgur.com/ICweZNK.png)

The log showed that dnsmasq forwarded the request to another DNS server.

This is expected behavior for a caching DNS service when it does not already have the requested information.

---

## 12. Observe DNS Caching

I repeated the same DNS query:

```bash
dig @localhost example.com
```

![11](https://i.imgur.com/PDNaBMt.png)

The second request produced the same DNS result.

However, the dnsmasq log showed different behavior.

![12](https://i.imgur.com/Eq40FXf.png)

Instead of forwarding the query again, dnsmasq used the information it had already cached.

### Key Concept

```text
First request
      ↓
dnsmasq
      ↓
Forward request to DNS server
      ↓
Receive response
      ↓
Cache result
```

Then:

```text
Second request
      ↓
dnsmasq
      ↓
Check cache
      ↓
Return cached result
```

---

## 13. Test a Nonexistent Domain

I also tested a domain name that did not exist.

![13](https://i.imgur.com/7ykSHJW.png)

dnsmasq did not have information for the domain, so it forwarded the query to the configured DNS server.

The response indicated that the domain did not exist.

![14](https://i.imgur.com/bA80F31.png)

---

## 14. Test DHCP with `dhclient`

After testing DNS, I tested the DHCP configuration.

The Linux DHCP client used in the lab was:

```bash
dhclient
```

The command was executed against:

```text
eth_cli
```

![15](https://i.imgur.com/bH8OdxO.png)

The lab used verbose/debug options to inspect the information received from the DHCP server.

---

## 15. Verify DHCP Information

The DHCP debugging output showed that the options configured in dnsmasq were correctly sent to the client.

![16](https://i.imgur.com/zUJbWnW.png)

The dnsmasq logs also showed that the server received the DHCP request and responded with an IP address.

---

## 16. Understand Hostnames and Network Interfaces

The lab demonstrated an important networking concept:

```text
Hostname ≠ Network Interface
```

For example:

```text
eth_cli
```

is the name of the network interface.

While:

```text
linux-instance
```

is the hostname of the machine.

DNS queries use hostnames rather than interface names.

The lab used:

```text
linux-instance.mycompany.local
```

to query the address associated with the client.

![17](https://i.imgur.com/A2M223r.png)

---

## 17. Configure Fixed DHCP Addresses

The company needed several servers to receive specific IP addresses.

The requested mappings were:

| MAC Address         | Assigned IP   |
| ------------------- | ------------- |
| `aa:bb:cc:dd:ee:b2` | `192.168.1.2` |
| `aa:bb:cc:dd:ee:c3` | `192.168.1.3` |
| `aa:bb:cc:dd:ee:d4` | `192.168.1.4` |

The lab used the `dhcp-host` configuration option to associate MAC addresses with specific IP addresses.

---

## 18. Modify the DHCP Dynamic Range

The lab required the first 20 IP addresses to remain outside the dynamic DHCP range.

The DHCP range was changed so that dynamic assignment started at:

```text
192.168.1.20
```

The DHCP lease duration was also changed from:

```text
24h
```

to:

```text
6h
```

The configuration was edited using:

```bash
sudo service dnsmasq stop
```

then:

```bash
sudo nano /etc/dnsmasq.d/mycompany.conf
```

---

## 19. Add the Fixed IP Configuration

The three `dhcp-host` entries were added to the configuration:

```text
dhcp-host=aa:bb:cc:dd:ee:b2,192.168.1.2
dhcp-host=aa:bb:cc:dd:ee:c3,192.168.1.3
dhcp-host=aa:bb:cc:dd:ee:d4,192.168.1.4
```

The dynamic range was changed to begin at:

```text
192.168.1.20
```

The lease time was changed to:

```text
6h
```

![18](https://i.imgur.com/n8fDAT4.png)

---

## 20. Validate the Updated Configuration

After making the changes, I checked the dnsmasq configuration again:

```bash
sudo dnsmasq --test
```

Expected result:

```text
dnsmasq: syntax check OK.
```

![19](https://i.imgur.com/XLVfawH.png)

This confirmed that the updated configuration had valid syntax.

---

## 21. Restart dnsmasq

After confirming the configuration:

```bash
sudo service dnsmasq start
```

The updated DHCP configuration was now active.

---

## 22. Test the Fixed IP Assignment

The lab used a simulated network environment to test the new fixed IP configuration.

The MAC address of `eth_cli` was temporarily changed to match one of the configured server MAC addresses.

The command used the:

```bash
ip link
```

command.

![20](https://i.imgur.com/HVEtN7Y.png)

This was done only within the simulated lab environment. The lab specifically warns against changing MAC addresses on actual production machines because it can create difficult networking problems.

---

## 23. Verify the Assigned IP

After changing the simulated interface configuration, the DHCP client requested an address.

The resulting DHCP traffic demonstrated that dnsmasq eventually responded with the requested fixed IP address.

The logs showed the DHCP process, including:

```text
DHCPREQUEST
```

and:

```text
DHCPDISCOVER
```

![21](https://i.imgur.com/gSpQXT6.png)

The final response confirmed that the desired address was assigned.

---
---

## Cybersecurity Relevance

Understanding DNS and DHCP is important for IT Support, Network Administration, and SOC Analyst work.

### DNS Monitoring

DNS activity can provide useful information about:

* Which domains systems are requesting
* DNS resolution failures
* Unexpected domain requests
* Network communication patterns

### DHCP Monitoring

DHCP information can help associate:

```text
MAC Address → Host → IP Address
```

This relationship can be useful when investigating devices on a network.

### Configuration Management

Incorrect DNS or DHCP configuration can cause:

* Connectivity problems
* Incorrect IP assignments
* DNS resolution failures
* Service availability problems

### Log Analysis

The lab also provided hands-on experience using service logs to understand what DNS and DHCP services are doing.

This is directly relevant to troubleshooting and security monitoring workflows.

---
---

## Key Concepts

| Concept     | Description                                                      |
| ----------- | ---------------------------------------------------------------- |
| DNS         | Resolves hostnames to IP addresses                               |
| DHCP        | Dynamically provides network configuration to clients            |
| dnsmasq     | Provides DNS forwarding/caching and DHCP services                |
| `dig`       | Performs DNS queries                                             |
| `dhclient`  | Requests network configuration through DHCP                      |
| MAC Address | Hardware/network interface identifier used for DHCP host mapping |
| DHCP Range  | Pool of addresses available for dynamic assignment               |
| DHCP Lease  | Period for which an assigned IP remains valid                    |
| DNS Cache   | Stores previously resolved DNS information                       |
| `ip`        | Linux command for inspecting/configuring network interfaces      |
| `service`   | Used to manage Linux services                                    |
| `nano`      | Terminal-based text editor                                       |

---
---

## Troubleshooting Workflow

A basic dnsmasq troubleshooting workflow from this lab is:

```text
Check network interface
        ↓
Check dnsmasq status
        ↓
Review configuration
        ↓
Enable logging
        ↓
Validate configuration
        ↓
Restart dnsmasq
        ↓
Test DNS
        ↓
Review DNS logs
        ↓
Test DHCP
        ↓
Review DHCP logs
        ↓
Verify IP assignment
```

---
---

## Important Commands

### Check network configuration

```bash
ip addr
```

### Check a specific interface

```bash
ip addr show eth_srv
```

### View dnsmasq configuration

```bash
cat /etc/dnsmasq.d/mycompany.conf
```

### Edit configuration

```bash
sudo nano /etc/dnsmasq.d/mycompany.conf
```

### Check service status

```bash
sudo service dnsmasq status
```

### Stop dnsmasq

```bash
sudo service dnsmasq stop
```

### Start dnsmasq

```bash
sudo service dnsmasq start
```

### Validate configuration

```bash
sudo dnsmasq --test
```

### Test DNS

```bash
dig @localhost example.com
```

### Request DHCP configuration

```bash
sudo dhclient eth_cli
```

### Inspect network interface configuration

```bash
ip link
```

---
---

## DNS vs DHCP

| DNS                            | DHCP                            |
| ------------------------------ | ------------------------------- |
| Resolves names to IP addresses | Assigns IP addresses            |
| Uses hostname information      | Uses client/network information |
| Tested with `dig`              | Tested with `dhclient`          |
| Can cache DNS responses        | Maintains DHCP leases           |
| Helps locate network services  | Helps configure network clients |

---
---

## Skills Demonstrated

* DNS Administration
* DHCP Administration
* Linux Networking
* Network Interface Management
* dnsmasq Configuration
* DNS Troubleshooting
* DHCP Troubleshooting
* IP Address Management
* MAC Address-Based DHCP Assignment
* Network Service Management
* Log Analysis
* Configuration Validation
* Linux Command Line
* Basic Network Security Monitoring

---
---

## Final Takeaway

This lab gave me hands-on experience configuring and troubleshooting **DNS and DHCP services with dnsmasq** in a simulated Linux network.

I practiced inspecting network interfaces, managing the dnsmasq service, modifying configuration files, validating configuration syntax, testing DNS resolution, observing DNS caching, requesting DHCP addresses, and assigning fixed IP addresses based on MAC addresses.

The lab also strengthened my understanding of how **DNS, DHCP, IP addresses, MAC addresses, network interfaces, and service logs** work together in a network environment.
