#### Trusting devices

This chapter emphasizes that **devices are the foundational battleground for security**. Since a compromised device cannot be trusted to report its own status, Zero Trust requires robust mechanisms to establish, maintain, and verify device trust.

### 1. Establishing Trust
*   **Bootstrapping:** Inherited manufacturer trust is insufficient. You must "inject" trust into hardware.
*   **Golden Images:** Always load a known-good, validated image onto new devices. This provides a baseline of software integrity.
*   **Secure Boot:** Use secure boot processes to validate firmware and OS loader signatures, preventing malicious persistence at low levels.

### 2. Identity and Key Security
*   **Device Certificates:** Use unique, CA-signed certificates for identification and to match devices against your inventory.
*   **Secure Storage:** Never store private keys in unprotected files or rely on simple OS permissions. **The gold standard is using secure cryptoprocessors (HSMs or TPMs)**, where the private key is generated and stored in hardware and never leaves the device.

### 3. Provisioning and Signing
*   **Human-in-the-loop:** For static infrastructure, manual authorization is most secure. To balance security and efficiency, use **Time-based One-Time Passwords (TOTP)**. Humans should only approve requests they initiated to prevent fatigue-related errors. A TOTP failure is a significant security event.

```
       [ Human ]
           |
           | (1) Provides TOTP
           v
       [ Provisioning Service ]
           |
           | (2) Forwards request + TOTP
           v
       [ Signing Service ] <-----> [ Verification ]
           |
           | (3) Signed Certificate
           v
       [ New Device ]
```

*   **Automated Provisioning:** In dynamic, auto-scaling environments, human authorization is not feasible. Trust must be sourced from:
    *   **The Resource Manager:** A privileged system that can assert, "I turned this host on." (Leverage cloud-native identities like service accounts/managed identities).
    *   **The Image/Device:** Using pre-baked credentials or hardware-backed keys (e.g., TPM).
*   **Multifaceted Authorization (The "Split Responsibility" Model):** To safely remove humans from the loop, **never rely on a single source**. The most secure approach requires multiple, disparate components.

```
       [ Resource Manager ]        [ New Device / Image ]
                |                            |
                | (1) Request                | (2) TPM/Image Key
                v                            v
           [ Signing Service ] <-------------+
           (Checks factors: RM + TPM + IP + Cert Props)
                |
                | (3) Validates & Signs
                v
           [ Issues Cert ]
```

By splitting these responsibilities, you ensure no single compromised component—such as a stolen image or an attacker-controlled resource manager—can independently grant unauthorized access.


### 5. Authenticating Devices with the Control Plane

To validate device identity over a network, Zero Trust relies on robust open standards and hardware-backed security.

#### X.509: The Identity Standard
**X.509** defines the format for public key certificates and validation methods for certification chains.
*   **Certificate Authorities (CA) & PKI:** A **CA** endorses certificate validity via digital signatures. Trusting the CA implies trusting all certificates in the resulting chain. A **Registration Authority (RA)** ensures details are accurate before signing.
*   **Asymmetric Cryptography:** X.509 utilizes a public/private key pair. The owner proves identity by performing cryptographic operations that only their private key can produce.
*   **Limitations:** While useful for metadata (via extensions) and enabling encrypted communication, the **private key** is often software-based, making it vulnerable to theft if stored on a disk.

#### HSMs & TPMs
**Hardware Security Modules (HSMs)** and **Trusted Platform Modules (TPMs)**—specialized cryptoprocessors—are the solution to protecting private keys.
*   **TPM Functionality:** A TPM generates and stores a **Storage Root Key (SRK)**, which serves as the trust root. To secure large volumes of data efficiently, TPMs employ **envelope encryption**. This is a hybrid cryptographic approach:
    1.  **Bulk Data Encryption (Symmetric):** The data is encrypted using a fast symmetric key (e.g., **AES**).
    2.  **Key Wrapping (Asymmetric/PKI-style):** The AES key itself is then "wrapped" (encrypted) by the asymmetric SRK.
    This binds the data to the specific hardware TPM, ensuring it can only be decrypted by the device that originally protected it, combining the speed of symmetric encryption with the robust security of asymmetric key protection.

```
       [ Bulk Data ]
             |
             | (1) Encrypt with AES Key (Symmetric)
             v
      [ Encrypted Data ]

       [ AES Key ]
             |
             | (2) Wrap with TPM's SRK (Asymmetric/PKI-style)
             v
     [ Wrapped AES Key ]
```
*   **Platform Configuration Registers (PCRs):** PCRs store hashes of system states (BIOS, boot records). Data can be **"sealed"** to specific PCR values, ensuring sensitive keys are only unlocked when the system is in a known, approved configuration.
*   **Remote Attestation:** Using the unique **Endorsement Key (EK)** and signed quotes of current PCRs, a TPM can prove both host identity and software state to a remote party.

#### Security Risks and Mitigations
Despite hardware protections, TPMs/HSMs are susceptible to sophisticated attack vectors:
*   **Attacks:** **ROCA**, **Side-Channel** attacks, and **Fault Injection** can bypass security by targeting shared secrets or exploiting physical/mathematical vulnerabilities.
*   **Mitigations:** Protection strategies include **Confidential Computing**, **Secure Boot**, regular patching, independent audits, and physical/logical security measures.

#### Supporting Legacy Devices
For legacy infrastructure that cannot host modern security agents, we move the Zero Trust termination point as close to the device as possible:
*   **Hardware Supplicant:** Instead of an application proxy, use a dedicated hardware device equipped with a TPM chip that plugs directly into the legacy host (e.g., **SCADA** or **HVAC** systems) to act as a secure Zero Trust supplicant.




