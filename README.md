Content-validated decentralized public key certificate issuance system

Han Zijun

cv_dcis@163.com

version: 0.1.0

Feb 24, 2026

www.github.com/Block-Genesis/CV-DCIS

Abstract:​ A truly decentralized public key certificate issuance protocol should be based on proofs of objective content rather than relying on assertions from third parties. We propose a solution for decentralized public key certificate issuance based on blockchain [1] and hash functions. The server submits the hash value of unpublished webpage content and the website's public key, which are then bound into a certificate by validation nodes and recorded on the chain. Validation nodes only accept hash values that have not previously appeared on the blockchain to prevent attackers from tampering with public keys. When a certificate expires and needs renewal or the owner needs to change key pairs, the old certificate's private key is used to sign the new certificate's public key, and the new certificate is recorded on the chain. Nodes only accept public key data corresponding to the latest blockchain state. When a client accesses a website, the server sends the relevant webpage code and the index location of the certificate on the blockchain to the client. The client hashes the code and compares it with the hash value in the certificate at the specified index. If they match, the client uses the public key in the certificate to establish a secure connection with the server.

1. Introduction

1.1 Current State and Dilemma of the Internet Trust System:​ Today's secure internet communication heavily relies on the Public Key Infrastructure (PKI). Undoubtedly, Certificate Authorities (CAs), as the core of PKI, provide a fast and efficient solution for establishing secure internet connections. However, this highly centralized system suffers from severe drawbacks such as single points of failure and censorship risks [2]. Existing solutions propose decentralization based on Proof of Authority (PoA) [3][4], but their essence remains centralized decision-making by authorities, not true decentralization [5].

1.2 A New Paradigm Based on Content Validation:​ CV-DCIS (Content-validated decentralized public key certificate issuance system) proposes a method that utilizes blockchain technology to directly pair the webpage content viewed by the client with the public key provided by the webpage provider, without the need for any third-party assertions. Here, the blockchain itself only records the association between webpage content and public keys. Validation nodes only maintain this relationship table and timestamps.

2. System Design

2.1 Design Goals and Core Principles:​ All designs of CV-DCIS must be based on decentralization. We are even willing to sacrifice some speed and convenience for a high degree of decentralization. Therefore, the goal of CV-DCIS is not to completely replace the traditional PKI/CA system. Its core objective is to provide a preliminarily viable solution for public key certificate issuance for Web3 and decentralized internet services.

2.2 System Overview and Key Roles:​ There are at least three roles in this system: the server applying for a certificate, the client checking the certificate when browsing webpages, and the validators (miners) responsible for recording certificates on the chain. It is important to clarify that the "validator" here is not a traditional certifier. The validator itself does not make any promises about the content; it is only responsible for recording objective content on the chain. There are no trusted third parties in this system. All nodes can directly verify the legitimacy of certificates based on the blockchain.

2.3 Core Workflow:​ When a new webpage appears on the server, it should broadcast the hash value of the webpage code and the website's public key to the blockchain before sending it to any client for the first time, and wait for validators to record it on the chain. After this certificate is correctly written to the blockchain, it is considered a legitimate certificate. The client should store block headers locally and update them frequently. When the client accesses the webpage, the server returns the relevant webpage code, public key, and the path of the certificate under the Merkle root in the block header. The client searches locally. If a legitimate certificate exists, the client compares the hash value of the public key in the certificate with the public key sent by the website. If they match, the client uses that public key.

3. Key Mechanisms and Security

3.1 Content-Public Key Binding and Certificate Lifecycle:​ We stipulate that one public key can be bound to hash values of multiple webpage contents, but the hash value of one webpage content can only be bound to one public key. Therefore, each webpage content hash value has one and only one public key paired with it. We define a piece of data consisting of a webpage content hash value and a public key hash value as a new certificate. One or more certificates are packaged into a block. The blockchain grows blocks at a fast and stable rate. Here, each certificate does not need an independent issuance date; its issuance date is directly set as the timestamp of the block it is packaged into.

3.2 Legitimate Certificate:​ A certificate is considered legitimate only if its webpage content hash value appears on the blockchain for the first time. Any attempt to create a certificate using a webpage content hash value already recorded on the blockchain is not allowed. This design effectively prevents attackers from obtaining webpage content and then applying for a certificate to replace the server's certificate. Therefore, the server should apply for a certificate beforepublishing the content.

3.3 Key Security Rotation Mechanism:​ Since the same webpage content hash value cannot appear repeatedly on the blockchain, when the server wants to change its key pair or when a certificate expires and a new one needs to be applied for, it would have to change the webpage content, which is obviously impractical. We define another legitimate certificate format that can keep the webpage content hash value unchanged. The public key part uses the old certificate's private key to sign the new certificate's public key, allowing the key pair to be changed without altering the webpage content.

3.4 Possible Attack Vectors:

(1) Collusion Attack:​ An attacker colludes with a validator (miner) to replace the public key when the server applies for a certificate, binding the webpage content hash value to the attacker's public key on the chain. This attack is technically feasible but completely impractical in application. First, the server only provides hash values (content hash and public key hash) to the validator. This information itself is anonymous, making it difficult to associate with a specific entity. Therefore, it is unrealistic for an attacker to launch targeted attacks against a specific entity. Second, even if the validator does bind the webpage content hash value to the attacker's public key, the server only needs to abandon that certificate, make a non-substantive modification to the webpage content (e.g., adding a comment that does not affect display), generate a new hash value, and reapply for a certificate.

(2) Preemptive Registration Attack:​ If an attacker has a method to obtain the webpage content or its hash value beforethe server applies for a certificate, they could theoretically apply for the legitimate certificate for that webpage content. Any subsequent certificate application by the server would be considered invalid. Therefore, the server must properly safeguard the webpage content before applying for a certificate. Even if an attacker successfully registers a certificate preemptively, the server can remediate by abandoning the current version of the webpage content and using a new version.

4. Decentralized Certificate Revocation List

4.1 Reason for Use:​ From the previous design, we can see the importance of the private key corresponding to a certificate in this system. If the private key is leaked, all certificates bound to the public key corresponding to that private key become theoretically insecure. The traditional solution generally involves introducing a trusted central authority to publish a Certificate Revocation List (CRL). However, since all designs of CV-DCIS must be based on decentralization, we introduce a decentralized certificate revocation list for this purpose.

4.2 Revocation Process and Optimization:​ We design a revocation list, similar in structure to a blockchain, specifically for recording revoked certificates. The revocation method involves signing the certificate with its private key and recording it on the chain. This design provides a way for certificate owners to revoke their own certificates without any third party. As the number of revoked certificates increases, the size of the blockchain may grow significantly. We introduce a pruning mechanism. Because certificates whose validity period has expired are revoked by default, we can remove these certificates from the revocation list. Thus, with certificates being added to and removed from the revocation list daily, a dynamic balance in size can be achieved.

4.3 Response Measures for Private Key Leakage:​ If an attacker obtains a certificate's private key, their possible attack method regarding the certificate is to sign a new certificate with the private key to escape the server's control over the certificate. For this purpose, we stipulate that within 24 hours after a new certificate is issued, the old certificate's private key can be used to revoke the new certificate. The server's operation should be to immediately use the old certificate's private key to revoke the new certificate, and then use the old certificate's private key to revoke the old certificate itself.

5. Comparison, Applications, and Prospects

5.1 Comparison with Traditional and Existing Solutions:​ Compared to the traditional centralized Public Key Infrastructure (PKI) system, the CV-DCIS solution has fundamental differences in its core trust model. Traditional PKI relies on one or more trusted Certificate Authorities (CAs) as trust anchors, and its security is based on trust in the CA entity and its security practices. This model has single points of failure and faces potential censorship and centralized control issues. CV-DCIS completely eliminates reliance on any trusted third party. Its trust foundation is built on the objective immutability of cryptographic hash functions and the decentralized ledger (blockchain). The legitimacy of a certificate does not originate from any institution's "assertion," but from the unique record on the public ledger of the objective fact that "a specific webpage content hash value is bound to a specific public key for the first time."

5.2 Potential Application Scenarios:​ The design goals of CV-DCIS make it naturally suitable for scenarios with strong demands for decentralization and censorship resistance, which can accept certain performance and complexity costs:

True Decentralized Web and Web3 Services:​ Provides a trustless secure communication layer foundation for decentralized applications (dApps), content delivery in distributed storage networks, and decentralized identity/self-sovereign identity systems. For example, ensuring that the front-end code of a distributed website accessed by a user has not been tampered with.

Open-source Software Supply Chain and Content Integrity Verification:​ Maintainers of software projects or open-source libraries can bind public keys to the hash values of specific versions of their released software packages, firmware images, or documentation. After downloading, users can verify whether the content hash matches the record on the chain to confirm the authenticity and integrity of the content source, defending against supply chain attacks.

High-value Digital Assets and Notarization:​ In scenarios requiring the establishment of a binding between digital content (e.g., digital artwork, important documents) and a specific holder's public key at a specific point in time, this system can provide a decentralized "notarization" service based on the content itself.

Internet of Things (IoT) and Device Identity:​ In distributed IoT environments, provides a lightweight identity binding and authentication mechanism for massive devices without requiring centralized CA services. The hash value of a device's firmware can serve as its unique identity identifier bound to the device's public key.

Conclusion:​ This paper proposes a fully decentralized public key certificate issuance scheme, CV-DCIS, which achieves trustless operation without relying on any third-party assertions through content validation. By directly binding webpage content hashes with public keys on the blockchain, it ensures the objectivity and tamper-resistance of certificates. The core advantage of this system lies in eliminating the single points of failure and censorship risks inherent in the traditional PKI system, providing a viable foundational solution for secure communication in Web3 and decentralized internet services. However, the content-validated paradigm also introduces significant challenges: First, it requires the server to complete certificate registration beforepublishing content, introducing complexity in key rotation. Second, clients must bear additional overhead such as blockchain data synchronization and local verification, which may lead to delays in user experience. Finally, the system's availability depends on a healthy, attack-resistant blockchain network as the public ledger. Therefore, CV-DCIS is not intended to fully replace the traditional CA system but serves as a complementary security infrastructure for specific decentralized scenarios. Future work will focus on optimizing verification efficiency, designing lighter client protocols, and exploring practical applications of this system in fields such as IoT and distributed storage.

References
[1] Nakamoto S. Bitcoin: A Peer-to-Peer Electronic Cash System[EB/OL].2008
[2] Xia Lingling, Wang Qun, et al. Research on the Application of Blockchain in PKI Security[J]. Journal of Computer Research and Development, 2024.
[3] Wood G. Proof of Authority: A Reputation-Based Consensus Algorithm[EB/OL]. Ethereum Foundation, 2017.
[4] Ethereum Foundation. Proof-of-Authority (PoA)[EB/OL].2025
[5] De Angelis S, et al. PBFT vs Proof-of-Authority: Applying the CAP Theorem to Permissioned Blockchain[C]//Italian Conference on Cybersecurity. 2018

Appendix

A. Cryptographic Parameters and Core Algorithms
[1] The hash function used throughout this paper is the SHA256 algorithm, with a fixed output of 32 bytes.
[2] The Merkle tree in this paper uses a standard binary tree with a branching factor of 2.
[3] This paper does not impose requirements on the algorithm used for the certificate key pair, but a secure key generation algorithm should be used.

Certificate Generation Algorithm
Input:Webpage content C, server private key s_priv
Steps:
Calculate content hash H_c = Hash(C)
Derive public key S_pubfrom s_priv
Calculate public key hash H_pub = Hash(S_pub)
Construct raw certificate data Cert_raw = (type, H_c, H_pub).

Certificate Verification Algorithm (Client Side)
Input:Webpage content C' received from server, public key P', and the corresponding certificate index path on the blockchain.
Steps:
Retrieve the recorded legitimate certificate Cert_recorded = (H_c, H_pub)from the local blockchain state based on the index path.
Calculate the hash of the received content H'_c = Hash(C').
Verify H_c == H'_c.
Calculate the hash of the received public key H'_pub = Hash(P').
Verify H_pub == H'_pub.
Output:A boolean value.

Key Security Rotation Algorithm
Input:Old key pair (Old_priv, Old_pub), new public key New_pub, content hash value Old_hashof the old certificate.
Steps:
Use the old private key Old_privto sign the new public key New_pub:
Sig_rotation = Sign(Old_priv, New_pub)
Construct rotation certificate data Cert_rotation = (type, Old_hash, Sig_rotation)
Submit Cert_rotationto the chain.

(The typefield indicates whether the certificate is a new certificate or a rotation certificate. Rotation certificates allow the same content hash value to appear.)

B. Frequently Asked Questions (FAQ)

Q: How is a "legitimate certificate" generated and defined?
A: In CV-DCIS, the generation of a legitimate certificate must meet two key conditions: (1) It must be submitted by the server, binding a webpage content hash value that has neverappeared on the blockchain with its own public key, and then recorded on the chain by validation nodes. (2) A webpage content hash value can be bound to one and only one public key. The "issuance date" of the certificate is the timestamp of the block it is packaged into. Any operation attempting to rebind a hash value already existing on the chain to a new public key will be rejected, ensuring the uniqueness and tamper-resistance of the certificate.

Q: When should the server apply for a certificate?
A: The server mustcomplete the process of binding the webpage content hash value with the public key on the chain beforepublishing that webpage content to any client for the first time. This is key to preventing "preemptive registration attacks." If content is published before the certificate is obtained, an attacker might steal the content and register it preemptively, preventing the server from obtaining a legitimate certificate for the original content.

Q: How does the system prevent collusion between an attacker and validation nodes (miners) to replace the public key when the server applies for a certificate?
A: First, the server submits hash values (content hash and public key hash). This information itself is anonymous, making it difficult for attackers to launch precise attacks against specific targets. Second, even if an attack occurs, the server's remediation cost is low: it only needs to abandon the tampered certificate, make a non-substantive modification to the webpage content (e.g., adding a comment that does not affect display), generate a new hash value, and then reapply for a certificate.

Q: What additional verification tasks does the client need to undertake? How does this impact the user experience?
A: The client needs to (1) maintain and synchronize block header data of the blockchain; (2) when accessing a website, retrieve the corresponding legitimate certificate from the local blockchain state based on the index path provided by the server; (3) calculate the hash value of the received webpage content and compare it with the record on the chain; (4) verify the public key hash. This increases the computational and storage overhead on the client and may cause some delay due to blockchain data synchronization. This is a trade-off made to achieve the goal of decentralization.

Disclaimer and Acknowledgments

Disclaimer:​ This document aims to explain the technical concepts and design scheme of the CV-DCIS system, for academic research and technical discussion only. The author encourages scholars and researchers to follow the spirit of open source, conduct in-depth discussions on the content of this document, provide feedback, or contribute code. Any individual or entity may conduct follow-up research, development, or commercialization attempts based on the ideas described in this document. However, the author assumes no legal liability for any direct or indirect consequences arising therefrom, including but not limited to technical implementation risks, security vulnerabilities during application, or economic losses.

Acknowledgments:​ I sincerely thank all researchers who have made outstanding contributions in the fields of blockchain technology and cryptography. Their pioneering work is the foundation upon which this proposal was conceived and documented. I also pre-thank future researchers who may participate in discussions, provide valuable feedback, or contribute code to this project.

Disclaimer:​ This document aims to explain the technical concepts and design scheme of the CV-DCIS system, for academic research and technical discussion only. The author encourages scholars and researchers to follow the spirit of open source, conduct in-depth discussions on the content of this document, provide feedback, or contribute code. Any individual or entity may conduct follow-up research, development, or commercialization attempts based on the ideas described in this document. However, the author assumes no legal liability for any direct or indirect consequences arising therefrom, including but not limited to technical implementation risks, security vulnerabilities during application, or economic losses.

Acknowledgments:​ I sincerely thank all researchers who have made outstanding contributions in the fields of blockchain technology and cryptography. Their pioneering work is the foundation upon which this proposal was conceived and documented. I also pre-thank future researchers who may participate in discussions, provide valuable feedback, or contribute code to this project.
(The original white paper is in Chinese,the English version is for reference only)
