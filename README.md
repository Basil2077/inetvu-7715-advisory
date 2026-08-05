# Security Advisory: Multiple Vulnerabilities in C-COM iNetVu 7715 VSAT Antenna Control Unit

**Vendor:** C-COM Satellite Systems Inc.
**Product:** iNetVu 7715 VSAT Antenna Control Unit
**Affected versions:** software versions prior to 9.11.1.1
**Fixed version:** 9.11.1.1
**Published:** 2026-08-05
**CVE IDs:** TBA

**Researchers:** Arslan Mahmood, Basil Abdulrahman —  Saudi Aramco.

---

## Summary

Five vulnerabilities were identified in the web management interface of the C-COM iNetVu 7715 VSAT Antenna Control Unit. Two are rated Critical: an authentication mechanism that binds session state to the device rather than to the client, and a complete absence of authentication on the endpoints that drive antenna motion and RF transmission.

The iNetVu 7715 controls the pointing and transmit state of a satellite ground terminal. An attacker with network access to the management interface can move the antenna, stow or deploy it, initiate satellite acquisition, and enable transmit — without credentials. Beyond loss of the affected link, uncommanded antenna motion and uncontrolled transmit carry physical and RF-interference consequences that extend past the device itself.

All findings were reported to C-COM Satellite Systems on 3 November 2025 and remediated in software version 9.11.1.1, released 11 February 2026.

---

## Findings

### 1. Improper Authentication — Authentication state not bound to client session

**CWE-287** · **Critical** · CVSS 3.1 **8.1** — `AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H`

The web interface does not associate an authenticated session with the client that established it. While a legitimate administrator holds an active session, any other host able to reach the management interface is served that same authenticated session and is granted full administrative access without presenting credentials.

The attacker requires no credentials, no user interaction, and no knowledge of the administrator's session token — only network reachability to the interface during an administrative session. Consequences include administrative takeover, credential change locking out the legitimate operator, and denial of service against the antenna.

### 2. Missing Authentication for Critical Function

**CWE-306** · **Critical** · CVSS 3.1 **9.1** — `AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:H/A:H`

Privileged control functions are reachable and executable without any authentication or authorization check. Confirmed unauthenticated operations include:

- Stow antenna
- Deploy antenna
- Find satellite (acquisition)
- Manual antenna movement (elevation, azimuth, polarization)
- Platform testing (EL/AZ/PL sweep across operator-supplied angle ranges)
- **Enable transmit**

Each is a state-changing physical action on a satellite ground terminal. Repeated stow/deploy or manual slew commands disrupt or deny the satellite link. Unauthenticated transmit enable is the most severe: an attacker who can key the uplink without credentials can cause RF emissions outside the operator's control, with potential impact on the satellite transponder and on other users of the space segment.

> Scoring note: this is scored `S:U` for a conservative base score. An argument exists for `S:C` (CVSS 10.0) on the grounds that unauthenticated transmit enable produces impact beyond the vulnerable component's security scope.

### 3–5. Cross-Site Request Forgery

**CWE-352** · **High** · CVSS 3.1 **8.8** — `AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:H`

The interface implements no anti-CSRF control on state-changing requests. An attacker who induces an authenticated administrator to load attacker-controlled content can cause the administrator's browser to issue forged requests to the device. Three distinct impacts were confirmed:

| # | Affected function | Impact |
|---|---|---|
| 3 | Controller reset / restore factory defaults | Loss of configuration; denial of service |
| 4 | Satellite and GPS configuration | Antenna mispointing; loss of link |
| 5 | Device and platform configuration | Operational disruption |

### 6. Password Transmitted in URL Query String

**CWE-598** · **Medium** · CVSS 3.1 **5.9** — `AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:N/A:N`

Login and password-change requests submit the password as a GET query parameter. Credentials are consequently exposed in browser history, web server and proxy logs, and the `Referer` header on outbound navigation. The management interface is served over plaintext HTTP, so the credential is additionally exposed to any party able to observe network traffic to the device.

---

## Impact

The iNetVu 7715 is deployed on mobile and fixed VSAT terminals, including in remote industrial, maritime, energy, emergency-response, and broadcast environments where satellite connectivity is often the only link available. Findings 1 and 2 allow an unauthenticated attacker with network reachability to take administrative control of the terminal and to command the antenna and transmitter directly.

Operators should assume that any unit reachable from an untrusted network segment was fully exposed prior to 9.11.1.1.

## Mitigation

Upgrade to software version **9.11.1.1** or later. Contact C-COM Satellite Systems for firmware.


## Disclosure Timeline

| Date | Event |
|---|---|
| 22 Sep – 2 Oct 2025 | Testing conducted |
| 3 Nov 2025 | Findings reported to C-COM Satellite Systems |
| 9 Nov 2025 | Full report delivered to vendor |
| 13 Nov 2025 | Technical call with vendor engineering |
| 17 Nov 2025 | Vendor confirms remediation work underway |
| 22 Dec 2025 | Vendor provides updated firmware addressing authentication and password findings |
| 11 Feb 2026 | Vendor releases software version 9.11.1.1 containing fixes |

This advisory contains no proof-of-concept code or exploit detail. Technical specifics sufficient to reproduce these issues are withheld.
