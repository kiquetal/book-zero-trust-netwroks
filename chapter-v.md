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
*   **Automated Provisioning:** In dynamic, auto-scaling environments, human authorization is not feasible. Trust must be sourced from:
    *   **The Resource Manager:** A privileged system that can assert, "I turned this host on." (Leverage cloud-native identities like service accounts/managed identities).
    *   **The Image/Device:** Using pre-baked credentials or hardware-backed keys (e.g., TPM).
*   **Multifaceted Authorization (The "Split Responsibility" Model):** To safely remove humans from the loop, **never rely on a single source**. The most secure approach requires multiple, disparate components (e.g., Resource Manager validation + TPM key + Valid IP + Expected Certificate Properties). By splitting these responsibilities, you ensure no single compromised component—such as a stolen image or an attacker-controlled resource manager—can independently grant unauthorized access.

### 4. Strategic Caution
*   **Geopolitics:** Be wary of relying solely on external Certificate Authorities (CAs). Geopolitical tensions can impact their availability or trustworthiness, as demonstrated by sanctions limiting certificate renewals.




