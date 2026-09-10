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
*   **Human-in-the-loop:** For static infrastructure, manual authorization of certificate requests by a human is the most secure method. For efficiency, use TOTPs (Time-based One-Time Passwords) to authorize requests without unnecessary administrative overhead.
*   **Automated Provisioning:** In dynamic, auto-scaling environments, human authorization is not feasible. You must rely on a combination of signals to establish trust, such as:
    *   Verification from the **Resource Manager** (the system creating the host).
    *   Authentication material baked into the **Image/Device** (e.g., a registered TPM key).
    *   Environmental checks (e.g., verifying the IP address).
*   **Multifaceted Authorization:** Never rely on a single signal. Mitigate risks by requiring multiple, disparate components (e.g., Resource Manager + TPM key + Valid IP) to authenticate a certificate signing request.

### 4. Strategic Caution
*   **Geopolitics:** Be wary of relying solely on external Certificate Authorities (CAs). Geopolitical tensions can impact their availability or trustworthiness, as demonstrated by sanctions limiting certificate renewals.




