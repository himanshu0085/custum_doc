### Impact Assessment – devopenvpnvm (Azure Ubuntu OpenVPN VM)

| Control                                                 | Impact on OpenVPN VM                                     | Risk Level |
| ------------------------------------------------------- | -------------------------------------------------------- | ---------- |
| Ensure DCCP is disabled                                 | OpenVPN does not use DCCP                                | No Impact  |
| Ensure SCTP is disabled                                 | OpenVPN uses TCP/UDP, not SCTP                           | No Impact  |
| Ensure authentication required for single-user mode     | Affects only recovery/emergency boot mode                | Low        |
| Disable source routed packets (`accept_source_route=0`) | Recommended security setting, rarely used                | No Impact  |
| Enable martian packet logging (`log_martians=1`)        | Additional kernel logging only                           | Low        |
| Enable reverse path filtering (`rp_filter=1`)           | May affect VPN routing, split tunnels, asymmetric routes | Medium     |
| Ensure permissions on `/etc/cron.daily`                 | Security hardening only                                  | No Impact  |
| Ensure permissions on `/etc/cron.hourly`                | Security hardening only                                  | No Impact  |
| Ensure permissions on `/etc/cron.weekly`                | Security hardening only                                  | No Impact  |
| Ensure permissions on `/etc/cron.monthly`               | Security hardening only                                  | No Impact  |
| Ensure permissions on `/etc/cron.d`                     | Security hardening only                                  | No Impact  |
| Ensure permissions on `/etc/ssh/sshd_config`            | Security hardening only                                  | No Impact  |
| Ensure logger configuration files are restricted        | Security hardening only                                  | No Impact  |
| Ensure mounting of USB storage devices is disabled      | Azure VM has no physical USB devices                     | No Impact  |
| Restrict access to root account via `su`                | May affect administrators using `su`                     | Low        |
| `/etc/gshadow` permissions = 0400                       | Standard Linux hardening                                 | No Impact  |
| `/etc/gshadow-` permissions = 0400                      | Standard Linux hardening                                 | No Impact  |
| `/etc/shadow` permissions = 0400                        | Standard Linux hardening                                 | No Impact  |
| `/etc/shadow-` permissions = 0400                       | Standard Linux hardening                                 | No Impact  |
| Ensure default deny firewall policy                     | Can block SSH/OpenVPN traffic if rules missing           | High       |
| Ensure password creation requirements                   | Affects local user accounts only                         | Low        |
| Ensure failed login lockout                             | May lock administrator accounts                          | Medium     |
| Disable cramfs filesystem                               | Rarely used filesystem                                   | No Impact  |
| Disable freevxfs filesystem                             | Rarely used filesystem                                   | No Impact  |
| Disable hfs filesystem                                  | Rarely used filesystem                                   | No Impact  |
| Disable hfsplus filesystem                              | Rarely used filesystem                                   | No Impact  |
| Disable jffs2 filesystem                                | Rarely used filesystem                                   | No Impact  |
| Ensure TIPC is disabled                                 | OpenVPN does not use TIPC                                | No Impact  |
| Restrict core dumps                                     | May reduce troubleshooting capability                    | Low        |
| Ensure password reuse is limited                        | User account policy only                                 | Low        |
| Ensure minimum password age ≥ 7 days                    | User account policy only                                 | Low        |
| Ensure maximum password age is configured               | User account policy only                                 | Low        |
| Disable packet redirect sending                         | Recommended networking hardening                         | No Impact  |
| Remove unnecessary accounts                             | Verify Azure/OpenVPN service accounts first              | Medium     |
| Ensure auditd service is enabled                        | Small CPU/logging overhead                               | Low        |
| Disable RDS support                                     | OpenVPN does not use RDS                                 | No Impact  |
| Ensure bootloader permissions are configured            | Security hardening only                                  | No Impact  |
| Enable bootloader password protection                   | Affects console recovery operations                      | Low        |
| Set default UMASK to 077                                | May affect scripts and shared files                      | Medium     |
| Disable ICMP redirects (`accept_redirects=0`)           | Recommended networking hardening                         | No Impact  |

---

## Controls Requiring Special Attention

| Control                           | Why It Matters                                             | Risk   |
| --------------------------------- | ---------------------------------------------------------- | ------ |
| `net.ipv4.conf.all.rp_filter = 1` | Can drop valid VPN traffic in asymmetric routing scenarios | Medium |
| Default deny firewall policy      | Can immediately break OpenVPN and SSH access               | High   |
| UMASK 077                         | Can impact automation, scripts, and shared files           | Medium |
| Failed login lockout              | Can lock out administrators                                | Medium |
| Remove unnecessary accounts       | May break Azure monitoring/security extensions             | Medium |

---

## Recommended Implementation Categories

### Safe to Implement Immediately

* DCCP, SCTP, TIPC, RDS disable
* Filesystem module disable (cramfs, freevxfs, hfs, hfsplus, jffs2)
* Cron permissions
* SSH configuration permissions
* Shadow/gshadow permissions
* Source routing disable
* ICMP redirect disable
* Martian packet logging
* Auditd enablement
* Core dump restrictions
* Bootloader permissions
* USB storage disable

### Implement After Testing

* Reverse path filtering (`rp_filter`)
* Default deny firewall policy
* UMASK 077
* Failed login lockout
* Remove unnecessary accounts

### Highest Risk of Service Impact

| Priority | Control                      | Potential Impact             |
| -------- | ---------------------------- | ---------------------------- |
| 1        | Default deny firewall policy | VPN/SSH outage               |
| 2        | `rp_filter = 1`              | VPN routing failures         |
| 3        | Remove unnecessary accounts  | Azure agent/service failures |
| 4        | UMASK 077                    | Script/application issues    |
| 5        | Failed login lockout         | Admin access issues          |

### Azure-Specific Checks Before Implementation

| Check                    | Why                                   |
| ------------------------ | ------------------------------------- |
| OpenVPN service status   | Ensure service remains operational    |
| OpenVPN listening port   | Must be allowed through firewall      |
| NSG rules                | Must allow VPN and SSH traffic        |
| Host firewall rules      | Must align with NSG rules             |
| `net.ipv4.ip_forward=1`  | Required for many OpenVPN deployments |
| Azure extension accounts | Prevent accidental service disruption |

**Overall Assessment:** Approximately **85–90% of the controls can be applied with minimal risk** on `devopenvpnvm`. Focus validation and testing efforts on the five controls listed under **Highest Risk of Service Impact** before deployment.
