**No Magic Required**

*How Zero-Knowledge Trust actually works --- and why every piece has
been battle-tested for years*

February 2026

**The question you should be asking**

If someone tells you they have built a system where your secrets are
completely protected, nobody can access your data, the architecture
eliminates entire categories of attack, and the user experience is
simpler than what exists today --- you should be skeptical. The history
of security is littered with systems that sounded too good to be true
because they were.

So here is the honest answer to the honest question: how does
Zero-Knowledge Trust actually work? What makes it real and not snake
oil?

There is no trick. There is no novel cryptographic algorithm. There is
no breakthrough mathematical discovery. There is no magic.

There is an architecture. ZKT is a pattern --- a set of principles for
how cryptographic primitives should be assembled so that the user holds
the only keys to their digital life. VettID is an implementation of that
pattern, and the one we will use as a concrete example throughout this
document. But the pattern itself is open: any implementation that
follows the same principles, using the same battle-tested primitives,
would produce the same security properties.

The iPhone did not invent the touchscreen, the camera, or GPS. It
assembled them in a way that created something new. ZKT does the same
thing with cryptographic primitives. The innovation is the assembly, not
any individual piece.

> *Every piece of Zero-Knowledge Trust has been beaten on by the best
> cryptographers in the world for years. The innovation is not any
> individual piece. It is the assembly.*

**The building blocks**

Before we explain how the system works, here is everything it is built
from. This is the inventory for VettID's implementation specifically,
but any ZKT implementation would draw from the same pool of established
primitives. Every item on this list is a published, peer-reviewed,
widely deployed standard. None of them were invented by VettID.

  ------------------------ ------------------------ -----------------------------
  **Primitive**            **What it does**         **Who else uses it**

  **X25519**               Key exchange --- two     Signal, WhatsApp, WireGuard,
                           parties establish a      TLS 1.3, SSH, Cloudflare
                           shared secret without    
                           transmitting it          

  **XChaCha20-Poly1305**   Authenticated encryption Google, Cloudflare,
                           --- encrypts data and    1Password, NordVPN, WireGuard
                           proves it has not been   
                           tampered with            

  **Ed25519**              Digital signatures ---   SSH, Signal, Tor, Solana,
                           proves a message came    FIDO2/WebAuthn
                           from a specific party    

  **Argon2id**             Password hashing ---     Winner of the 2015 Password
                           makes brute force        Hashing Competition;
                           computationally          recommended by OWASP
                           infeasible (60 million   
                           times slower than SHA256 
                           on GPUs)                 

  **HKDF**                 Key derivation ---       TLS 1.3, Signal Protocol,
                           derives multiple         WireGuard
                           purpose-specific keys    
                           from a single secret     

  **TLS 1.3**              Transport encryption --- Every HTTPS website, every
                           secures data in transit  major browser, every cloud
                           between devices          provider

  **AWS Nitro Enclaves**   Hardware isolation ---   AWS production
                           creates a sealed compute infrastructure, Signal
                           environment even AWS     server-side processing
                           cannot access            

  **HMAC-SHA256**          Integrity verification   JWT, TLS, IPsec, every major
                           --- detects any          API authentication system
                           modification to stored   
                           data                     
  ------------------------ ------------------------ -----------------------------

That is the complete list. There is no secret ingredient. Every
algorithm, every protocol, every hardware component is published,
documented, and available for independent review.

**Built to evolve**

Because ZKT is an assembly of standard primitives rather than a
proprietary algorithm, it is designed to evolve as cryptography evolves.
When post-quantum cryptography (PQC) algorithms reach production
readiness --- lattice-based key exchange replacing X25519, for example
--- a ZKT implementation can swap in the new primitives without changing
the architecture. The pattern stays the same: encrypt, authenticate,
use, rotate, re-encrypt. Only the specific algorithms change.

This is a direct benefit of building on standards rather than invention.
A proprietary algorithm would require replacing the entire system when
quantum computing advances. A standards-based assembly replaces
individual components while the architecture continues to work. The
building blocks are modular by design.

**What the user actually does**

Before we explain the cryptography, here is what ZKT feels like to a
real user. We will use VettID's implementation as the example, but the
mental model applies to any ZKT system. Because if the security only
works in theory and the user experience is terrible, none of it matters.
The mental model has four steps.

**1. Enroll**

You install the VettID app and scan a QR code from your account portal.
The app walks you through creating a PIN and a password. Behind the
scenes, your vault is being provisioned inside a hardware-isolated
enclave, cryptographic keys are being generated, and your Protean
Credential is being created. You see none of this. You see: scan code,
create PIN, create password, done.

**2. Add data**

You add the things you want to protect: private keys, credentials, seed
phrases, personal information. These go into your vault, encrypted with
keys that only the enclave holds. Your phone stores an encrypted blob it
cannot open. The vault stores the rest in an encrypted database. You
manage what goes in through the app.

**3. Connect**

You connect to people and services. Each connection is a tendril --- a
scoped, revocable link between your vault and something outside it. You
control the scope, the duration, and the access level. Connections are
visible on your vine. You can see every active connection and sever any
of them at any time.

**4. Go**

That is it. Your vault handles the cryptography. Your vine manages the
connections. Your secrets are used when needed and never exposed. You
open the app, enter your PIN, and your digital life is at your
fingertips. When you need to authorize something sensitive, you enter
your password. The system works.

> *Enroll. Add data. Connect. Go. That is the entire user experience.
> Everything else is handled by the architecture.*

**Organize your vine, don't overload your root**

One critical design principle makes the system work well in practice:
keep your root lean. Your vault is a keychain, not a filing cabinet. It
holds the keys to your digital life, not the data itself. And your
connections should be organized thoughtfully, not piled on top of one
another.

> **What not to do:** Don't add every service, every contact, and every
> credential as a direct connection from the root. If your root has two
> hundred connections dangling off it, you have recreated the problem
> ZKT was designed to solve --- a flat, unmanageable tangle where
> everything has the same level of access and visibility.
>
> **What to do instead:** Think of your vine the way you think about
> files on a computer. You do not put every file on the desktop. You
> organize into folders. Your vine works the same way. Your root
> connects to a few high-trust branches: your financial identity, your
> professional identity, your personal connections. Each branch manages
> its own connections. Your bank tendril connects to financial services.
> Your professional tendril connects to work tools. Your personal
> tendril connects to family and friends.

This organization is not just aesthetics. It is security architecture. A
compromise at one branch does not propagate to another. A misbehaving
service connection on your professional branch cannot access your
financial branch. The structure of your vine is your security posture,
and a well-organized vine is a well-protected vine.

Your vault stores the cryptographic keys that control each branch. The
data --- your medical records, your financial accounts, your employment
history --- stays in the systems designed to manage it. Your vine
carries authority, not data. The root stays lean regardless of how many
connections you have, because the root holds keys, not content.

**How it actually works**

Now the machinery. We will walk through VettID's implementation step by
step, naming every algorithm at each stage. Other ZKT implementations
might choose different primitives from the same families --- a different
authenticated encryption cipher, a different key exchange protocol ---
but the pattern would be the same. No hand-waving.

**The Protean Credential**

Your Protean Credential is an encrypted package that lives on your
phone. It contains your identity keypair, your private keys, your seed
phrases, your password hash, and your policies. You hold this package,
but you cannot open it. It is encrypted with a Credential Encryption Key
(CEK) that only your vault holds.

> **Encryption:** The CEK is an X25519 keypair. The credential blob is
> encrypted using XChaCha20-Poly1305 authenticated encryption. The CEK
> private key --- the only thing that can decrypt the blob --- lives
> inside the vault's encrypted SQLite database, inside the Nitro
> Enclave. Both halves of the CEK keypair are held by the vault. Your
> phone never has the CEK private key. Your phone has the locked box.
> The vault has the key.

**Two factors, two purposes**

ZKT uses two separate authentication factors, each protecting a
different layer. They are not interchangeable.

> **Vault PIN (6 digits):** Entered when you open the app. The PIN is
> sent to the enclave supervisor, which combines it with sealed material
> from AWS KMS to derive the Data Encryption Key (DEK). The DEK decrypts
> the vault's SQLite database. Wrong PIN means wrong DEK, which means
> the database decryption fails cryptographically. The PIN is not
> compared against a stored value. It is verified by whether the math
> works. There is no hash to steal.
>
> **Credential password:** Entered for each sensitive operation (signing
> a transaction, using a key). The password is hashed with Argon2id on
> your device, encrypted with a single-use User Transaction Key (UTK),
> and sent to the vault. The vault decrypts it with the corresponding
> Ledger Transaction Key (LTK), compares the hash to the one stored
> inside your Protean Credential, and proceeds only if they match. The
> UTK/LTK pair is single-use --- destroyed after one operation --- so
> even if an attacker intercepts the encrypted hash, they cannot replay
> it.

  -------------- ------------------ ------------------ ------------------
                 **Vault PIN**      **Credential       **Why both?**
                                    Password**         

  **Protects**   Vault storage (DEK Operation          Compromising one
                 derivation)        authorization      does not
                                                       compromise the
                                                       other

  **When used**  Each session (app  Each sensitive     PIN is
                 open)              operation          convenience +
                                                       security; password
                                                       is per-action gate

  **Verified     Crypto failure     Hash comparison    PIN verified by
  by**           (wrong DEK =       (Argon2id)         hardware; password
                 decryption fails)                     verified by
                                                       software
  -------------- ------------------ ------------------ ------------------

**What happens when you use a secret**

You want to sign a Bitcoin transaction. Here is exactly what happens,
step by step, algorithm by algorithm.

> **Step 1.** Your app sends the encrypted Protean Credential blob and
> the operation request to the vault-manager inside the Nitro Enclave.
>
> **Step 2.** The vault-manager decrypts the credential using the CEK
> private key (X25519 + XChaCha20-Poly1305). Your secrets now exist in
> hardware-isolated enclave memory. Nowhere else.
>
> **Step 3.** The vault challenges you: enter your credential password.
> It tells the app which UTK to use for encrypting the response.
>
> **Step 4.** You enter your password. The app hashes it with Argon2id
> (64 MB memory cost, 3 iterations, parallelism 4) and encrypts the hash
> with the specified UTK public key (X25519). The encrypted hash is sent
> to the vault.
>
> **Step 5.** The vault decrypts the hash with the corresponding LTK
> private key. The LTK is single-use: it is destroyed immediately after
> decryption. The vault compares the received hash against the hash
> stored in the credential. If they match, you are authorized.
>
> **Step 6.** The vault loads your Bitcoin private key from the
> credential, signs the transaction (secp256k1), and immediately zeros
> the key from memory. The key existed in enclave memory for
> milliseconds.
>
> **Step 7 --- the protean part.** The vault generates a brand new CEK
> keypair (X25519). It re-encrypts your entire credential with the new
> CEK public key. It stores the new CEK private key in the DEK-encrypted
> SQLite database and syncs to S3. It returns the re-encrypted
> credential blob, the transaction signature, and fresh UTKs to your
> app.
>
> **Step 8.** Your app stores the new encrypted blob and the new UTKs.
> The old CEK is dead. The old blob is cryptographic garbage. If anyone
> intercepted it, they have nothing usable.
>
> *The credential changes shape after every single use. That is why it
> is called protean. There is no stable key to target. An attacker who
> steals today's blob has a package locked with a key that was destroyed
> the moment the operation completed.*

X25519 key generation takes approximately 0.05 milliseconds. That is why
rotation after every operation is practical. There is no performance
penalty for changing every lock in the building after every use, because
the locks are nearly free to manufacture.

**The hardware guarantee**

The obvious question: what if VettID is the attacker? What if the
company that runs the vault modifies the code to exfiltrate secrets?
What prevents a rogue employee, a compromised server, or a government
subpoena from accessing your data?

The answer is hardware, not trust. A ZKT implementation requires a
hardware-isolated environment where secrets can be processed without
exposure. VettID uses AWS Nitro Enclaves. Other implementations might
use different trusted execution environments --- the principle is the
same. The secrets must be processed inside a sealed compute environment
that even the operator cannot access.

A Nitro Enclave is a hardware-isolated virtual machine with three
critical properties: no persistent storage, no external network access,
and no interactive access --- not even by AWS administrators. The
enclave communicates with the outside world through a single constrained
channel (vsock), and everything passing through that channel is
encrypted.

**Attestation: proving the code is honest**

When your app connects to the vault, it does not take VettID's word that
the correct code is running. It verifies cryptographically.

> **How attestation works:** The enclave's Nitro Security Module
> (hardware, not software) generates an attestation document containing
> PCR values --- cryptographic hashes of the exact code running inside
> the enclave. Your app sends a random nonce, which is included in the
> attestation document to prevent replay. The app verifies the AWS
> signature on the attestation, checks the PCR values against VettID's
> published values, and confirms the nonce matches. If any check fails,
> the app refuses to proceed.

This is not a policy. It is physics. The Nitro hardware generates the
attestation. AWS cannot forge it. VettID cannot forge it. If the code
has been modified by anyone for any reason, the PCR values change, and
every app in the world will reject the connection.

**Sealed storage: the code protects the data**

Your vault's encryption material is sealed to the enclave's code
identity using AWS KMS. The sealed material can only be unsealed by a
Nitro Enclave running the exact same code (matching PCR values). If
VettID deploys different code --- even a one-byte change --- the sealed
material cannot be unsealed. The vault data becomes inaccessible.

> **What this means:** VettID cannot deploy a modified enclave that
> exfiltrates data, because the modified enclave cannot unseal the
> existing vault data. The PCR values would differ. The KMS policy would
> reject the unseal request. The vault would refuse to start. Even the
> parent EC2 instance that hosts the enclave sees only encrypted blobs
> it cannot read. The math prevents exfiltration, not a policy document.

The parent process that manages network traffic and S3 storage
explicitly cannot access vault keys, plaintext data, or session keys. It
sees encrypted blobs and routing metadata. That is all.

**You approve enclave updates**

Enclave code changes over time --- bug fixes, security patches, new
features. When this happens, the new code has different PCR values. Your
vault's sealed material is bound to the old PCR values and cannot be
unsealed by the new code. This is a feature, not a bug. It means nobody
can silently swap the code your vault runs.

In VettID's implementation, the process is transparent: when a new
enclave version is deployed, your app is notified. The old enclave and
the new enclave run simultaneously. You are not moved to the new version
until you approve. When you do, your sealed material is re-sealed for
the new PCR values inside the old enclave (which can still unseal it),
and then verified on the new enclave before cutover. The old version
stays available for rollback.

> **Why this matters:** In the current model, service providers push
> server-side changes constantly and you have no visibility or control.
> You trust that every update is benign. In ZKT, a code change produces
> different PCR values, your app can detect it, and you decide when to
> move. The attestation chain makes invisible changes impossible and
> gives users meaningful control over the software that handles their
> secrets.

**Compared to what?**

ZKT sounds ambitious until you compare it to what currently passes for
security. In the current model, your passwords exist in databases
operated by hundreds of companies, each with hundreds of employees who
have database access, each running infrastructure with thousands of
potential vulnerabilities. A single breach at any one of these companies
exposes your credentials. This is not a theoretical risk. It is a weekly
news story.

ZKT's claim is actually more modest than the current model's implicit
claim.

  ----------------------------------- -----------------------------------
  **Current model claims**            **ZKT claims**

  Hundreds of employees with database The hardware enclave is isolated
  access will never make a mistake or and the math is correct
  act maliciously                     

  Every server will remain            Even a compromised server sees only
  uncompromised indefinitely          encrypted blobs it cannot read

  Security policies will be followed  Architectural properties hold
  perfectly by every person in every  regardless of who follows what
  organization                        policy

  Bearer tokens will not be           Credentials change after every use;
  intercepted or stolen               stolen credentials decrypt with
                                      dead keys
  ----------------------------------- -----------------------------------

The current model requires trusting thousands of people and systems to
behave correctly, indefinitely. ZKT requires trusting published
algorithms and hardware isolation. One of these is a human problem with
no known solution. The other is a mathematics problem with decades of
validation.

> *ZKT does not claim to be perfectly secure. It claims to reduce the
> trust surface from "every person and system that touches your data" to
> "the math and the hardware." That is not a bigger claim. It is a
> dramatically smaller one.*

**What could go wrong**

No security system is perfect. Honest engineering requires acknowledging
attack surfaces. Here is what we know about ZKT's.

> **Device compromise:** If malware captures your PIN and password as
> you type them, and exfiltrates your encrypted credential blob before
> you use it, an attacker could race you to the vault. This is mitigated
> by device attestation (the vault verifies your device has not been
> rooted or tampered with) and by the single-credential design: the
> first person to use the credential rotates the CEK, making the other
> copy worthless. But a sophisticated, targeted device attack remains a
> risk.
>
> **Enclave vulnerability:** If a vulnerability is discovered in the
> Nitro Enclave hardware itself --- not the code, but the isolation
> boundary --- the security model would be affected. This is a risk
> shared by every system that relies on hardware security: Intel SGX,
> ARM TrustZone, Apple's Secure Enclave. Nitro has a strong track
> record, but no hardware is perfect.
>
> **Lost device:** A ZKT implementation must address device loss. VettID
> handles this with a hosted backup: every time the vault issues a new
> credential blob to your phone, it simultaneously stores a copy in
> VettID's backend. This backup is the same encrypted blob --- it is
> useless without a genuine Nitro Enclave and your PIN. If you lose your
> device, you log into the account portal, request recovery, and wait 24
> hours (an intentional delay that gives you time to cancel if the
> request was unauthorized). After the waiting period, you scan a QR
> code on your new device and enter your PIN. The backup blob is
> delivered, the vault warms, and you are back. No seed phrase required.
> Other ZKT implementations might handle backup differently, but the
> principle is the same: the backup is encrypted material that cannot be
> used without the user's authentication factors and a trusted execution
> environment.

These are real risks. They are also dramatically smaller and more
contained than the risks in the current model, where a single database
breach at a single company can expose millions of users simultaneously.
ZKT's failure modes are individual and targeted. The current model's
failure modes are systemic and catastrophic. And for the most common
risk --- losing a device --- the backup model means recovery is
straightforward, while the encrypted backup itself remains useless to
anyone who breaches the backend.

**The theft detection race**

One property of the single-credential design deserves its own
explanation, because it is counterintuitive and often mistaken for a
weakness. It is actually a deliberate security feature.

Because the credential changes after every use, there is only one valid
copy at any given moment. If an attacker steals your encrypted
credential blob, they have a narrow window: they must present it to the
vault and authenticate before you do. The moment either party uses the
credential, the CEK rotates and the other copy becomes garbage.

If you use your credential first (which is overwhelmingly likely, since
you use it routinely), the attacker's stolen copy is worthless. If the
attacker somehow uses it first, your next attempt fails, which is itself
a detection signal --- the vault knows something is wrong, because a
credential was used that should not have been. This is not a bug. It is
a tripwire.

> *In the current model, a stolen password works forever unless you
> change it, and you rarely know it has been stolen. In ZKT, a stolen
> credential works exactly once, and using it immediately alerts both
> the vault and the legitimate user.*

**The open challenge**

If the primitives are standard and the architecture is sound, the
appropriate response to skepticism is not persuasion. It is invitation.

Every cryptographic primitive used in ZKT is published. Every algorithm
is named. In VettID's implementation, the enclave code produces
verifiable PCR hashes that anyone can check. The architecture is
documented in detail. Any ZKT implementation worthy of the name should
be designed to be audited, not trusted on faith.

> **The invitation:** Review the primitives. Audit the code. Verify the
> attestation chain. Try to find the flaw. If the architecture is
> broken, we want to know. If a primitive has a weakness we have not
> accounted for, we want to know. If the assembly introduces a
> vulnerability that the individual components do not have, we want to
> know. The best security is security that has been attacked by smart
> people and survived.

We do not ask you to trust us. We ask you to verify. That is the entire
point of zero-knowledge trust: trust is replaced by verification. The
same principle that protects your data from VettID protects your
judgment from our marketing.

**One more time, simply**

Your vault is a keychain, not a filing cabinet. Your phone holds a
locked box. The vault holds the key. When you need your secrets, the
vault opens the box inside a hardware-sealed room, uses your secrets,
locks everything back up with a new key, and hands you the new locked
box. The old key is destroyed. Nobody else was in the room. Nobody can
get into the room. The room's walls are guaranteed by hardware, not
promises.

You enroll. You add your data. You connect to the world. You go. And you
organize your vine thoughtfully --- not every connection at the root,
but branches that reflect how you actually live your digital life.

Every piece of this has been tested for years. Every algorithm is
published. Every claim is verifiable.

> *Sufficiently well-assembled engineering is indistinguishable from
> magic. But it is not magic. It is X25519, XChaCha20-Poly1305,
> Argon2id, hardware enclaves, HKDF, and Ed25519 --- arranged so that no
> one except you can access your digital life. VettID is one
> implementation. The pattern is open. The pieces are boring. The
> assembly is the breakthrough.*
