**Three Locks**

*Not everything in the vault is equal. The guardian knows the
difference.*

February 2026

**Not everything is equal**

Your vault holds your name and it holds your Bitcoin private key. These
are not the same thing. Leaking your name is inconvenient. Leaking your
private key is catastrophic and irreversible. If the vault treated both
the same way --- same protections, same access rules, same
authentication --- it would either over-protect the trivial or
under-protect the critical.

The vault does not treat them the same way. Everything in your vault
belongs to one of three tiers, and each tier has its own lock. The
guardian knows which tier holds what, and the guardian enforces the
right protection for each one. You do not need to manage this. But it
helps to understand what the guardian is doing and why.

> *Your name and your private key both live in the vault. They do not
> get the same lock. Three tiers of data, three levels of protection,
> and the guardian enforces the right one on every request.*

**The first lock: personal data**

Personal data is the information that makes up your profile: your name,
your email address, your phone number, your mailing address. The things
that connections request through contracts. Important enough to protect,
but not catastrophic if exposed.

Personal data lives in the vault's encrypted database, inside the
personal namespace. When you open the app and enter your PIN, the vault
warms: the PIN is combined with sealed hardware material to derive the
encryption key for the database, and the database becomes accessible to
the guardian. From that point on, the guardian can serve personal data
to connections through the normal contract enforcement pipeline ---
checking tiers, retention, rate limits --- without asking you for
additional authentication.

This is the lightest lock. Your PIN opened the vault. The contract
governs who can see what. The guardian checks the contract on every
request and logs everything. But you are not interrupted for every data
access. A retailer requesting your shipping address at checkout does not
trigger a password prompt. The contract says the field is on-demand, the
guardian checks the terms, the handler serves the data, and the event is
logged. You set the rules when you accepted the contract. The guardian
enforces them without bothering you.

| **Property** | **Personal data** |
|---|---|
| **Where it lives** | Vault's DEK-encrypted database, personal namespace |
| **Authentication** | PIN to warm the vault. No per-operation password. |
| **Access control** | Contract enforcement: field tiers, retention, rate limits, consent where declared |
| **Examples** | Name, email, phone, mailing address, date of birth, profile photo |
| **Impact if exposed** | Privacy violation, identity risk, inconvenience. Serious, but recoverable. |

> *Personal data gets the first lock: the PIN that warms the vault.
> After that, the contract governs access and the guardian enforces the
> contract. You set the rules once. The guardian runs them
> continuously.*

**The second lock: secrets**

Secrets are the operational material that makes your connections work:
the keys that encrypt your messages, the credentials that authenticate
you to services, the tokens that maintain your sessions, the
authentication material that lives on the rings of your keychain. This
tier also includes identity and financial credentials like driver's
licenses, passports, and credit cards --- things that act on your behalf
rather than simply describing you.

Secrets also live in the vault's encrypted database, but in their own
namespaces. Connection keys live in the connection namespace. Service
credentials live in the service namespace. Each is isolated from the
others --- a retailer's ring cannot see your employer's ring, and
neither can see your personal namespace.

Like personal data, secrets are accessible after you enter your PIN. But
secrets are typically the things your connections want access to, which
means they should be gated. When a retailer needs your credit card to
process a payment, you should be asked. When an identity verification
service needs your driver's license, you should be asked. These are the
items that connections request, and nearly every time they do, the
request should come to you for approval. This is where the consent tier
in the contract system earns its keep --- secrets are the natural home
for consent-level access, where the vault blocks the request until you
explicitly say yes.

The guardian serves secrets through the handler pipeline with contract
enforcement, and when the contract declares a secret as a consent field,
the flow pauses. The request arrives, your app shows you what is being
asked and by whom, and you approve or deny. The handler only fires after
your approval. This is not the password challenge that critical secrets
require --- it is a consent gate. You are not re-authenticating. You are
deciding whether to share.

The difference between personal data and secrets is not in how they are
protected --- both sit behind the same database encryption, the same
PIN, the same contract enforcement. The difference is in what they do.
Personal data describes you. Secrets act on your behalf --- a connection
key encrypts a message, a driver's license proves your identity, a
credit card authorizes a purchase. And the consequence of exposure is
different: a stolen credential can cause real damage, but the damage is
scoped. Cancel the card, report the license, rotate the key, and the
exposure ends. It is not pleasant, but it is contained and recoverable.

| **Property** | **Secrets** |
|---|---|
| **Where it lives** | Vault's DEK-encrypted database, connection and service namespaces |
| **Authentication** | PIN to warm the vault. No per-operation password. |
| **Access control** | Contract enforcement plus namespace isolation. Typically consent-gated: connections must ask, and you approve each time. |
| **Examples** | Connection encryption keys, service session tokens, authentication credentials, API tokens, driver's licenses, passports, credit cards |
| **Impact if exposed** | Compromised connection, service session, or credential. Damage is scoped and recoverable through rotation, revocation, or reissuance. |

> *Secrets get the second lock: the PIN, namespace isolation, and a
> consent gate. These are the things connections want access to --- your
> cards, your credentials, your licenses --- and you should be asked
> nearly every time. A stolen secret can cause real damage. Cancel it,
> rotate it, reissue it, and the damage ends.*

**The third lock: critical secrets**

Critical secrets are the things where a single exposure is permanently
devastating. Your Bitcoin private key. Your identity signing key. Your
financial authorization credential. Your seed phrases. These are not
recoverable by rotation. If someone gets your private key, they can
drain your wallet. If someone gets your signing key, they can
impersonate you. The damage is immediate, total, and in many cases
irreversible.

Critical secrets do not live in the vault's database. They live inside
the Protean Credential itself --- the encrypted blob on your phone that
only the guardian inside the tower can open. This is a fundamentally
different storage location with fundamentally different access rules.

Using a critical secret requires your password. Not just the PIN that
warmed the vault. The full password, hashed with Argon2id on your
device, encrypted with a single-use transaction key, sent to the
guardian, verified inside the enclave. And after the operation
completes, the guardian generates a new encryption key, re-encrypts the
entire credential, destroys the old key, and sends you the new locked
box. The critical secret existed in enclave memory for milliseconds. The
old credential is now cryptographic garbage.

**The window**

Requiring a password for every single critical operation sounds brutal.
If you need to sign three transactions in a row, entering your password
three times is friction that serves no security purpose --- you are
already authenticated, you are clearly present, and interrupting you
repeatedly adds frustration without adding safety.

This is where the window comes in. When you enter your password, you are
telling the guardian: it is me, and it is going to be me for the next
few minutes. You define the window --- the TTL. During that window, you
can use multiple critical secrets without being challenged again. The
guardian still receives the credential with each operation. The CEK
still rotates after each use. The audit trail still records every
access. The protean cycle never pauses. But the guardian does not demand
your password again until the window expires.

The window is yours to set. Cautious? Set it to thirty seconds.
Comfortable? Set it to five minutes. Paranoid? Set it to zero and
authenticate every time. The guardian respects your threshold. The point
is that you choose the balance between security and convenience, and the
choice is enforced consistently.

After the window expires, the next critical operation requires your
password again. No exceptions. No "remember me." No session cookie that
lingers indefinitely. The window closes and the third lock re-engages.

| **Property** | **Critical secrets** |
|---|---|
| **Where it lives** | Inside the Protean Credential (CEK-encrypted blob), not the database |
| **Authentication** | Password required. Argon2id hashed on device, encrypted with single-use transaction key, verified in enclave. User-defined TTL window for consecutive operations. |
| **Access control** | Credential decryption inside enclave only. CEK rotates after every operation. Old credential destroyed. |
| **Examples** | Private keys (Bitcoin, Ethereum), identity signing keys, financial authorization keys, seed phrases |
| **Impact if exposed** | Permanent and potentially catastrophic. Asset theft, identity compromise, irreversible damage. |

> *Critical secrets get the third lock: your password, verified inside
> the enclave, with a user-defined window for consecutive operations.
> The credential rotates after every use. The old key is destroyed.
> These are the secrets where a single exposure is catastrophic, and the
> guardian treats them accordingly.*

**Connections can set the lock**

Here is where the tier system becomes powerful beyond personal use. When
a connection adds something to your vault, it can declare which lock
applies.

A retailer that stores a loyalty token on your keychain ring adds it as
a secret. It sits in the connection namespace, accessible through the
normal handler pipeline, governed by the contract. The guardian serves
it when the retailer's systems need it. No password prompt. This is
appropriate --- a loyalty token is operationally useful but not
devastating if compromised.

A bank that provisions a transaction signing key through your connection
takes a different approach. The bank adds the signing key to your
Protean Credential as a critical secret. Now, every time that key is
used --- every time you authorize a transaction --- your vault requires
your password. The bank did not implement this protection. The bank
declared the sensitivity. Your vault enforced it. The signing key lives
inside the credential, subject to the full protean cycle: password
challenge, CEK rotation, old key destroyed.

The bank does not need to trust your vault's security model. The bank
declares "this is critical" and the architecture does the rest. The
protection is the same whether the critical secret was created by you or
provisioned by a connection.

**Use cases**

> **Retailer stores loyalty token → secret.** Lives in the connection
> namespace. Accessible through handlers after PIN. Governed by the
> contract. If compromised, someone gets your loyalty points. Annoying,
> not catastrophic.
>
> **Bank provisions transaction signing key → critical secret.** Lives
> inside the Protean Credential. Password required for every use (within
> TTL window). CEK rotates after each signing. If someone somehow
> obtained the encrypted blob, they cannot use the signing key without
> your password and a genuine enclave.
>
> **Healthcare provider adds consent credential → critical secret.**
> Every time this credential is used to authorize access to your medical
> records, you enter your password. The provider declared the
> sensitivity. Your vault enforces it. Nobody accesses your health data
> without your active, authenticated, audited consent.
>
> **Employer provisions VPN certificate → secret.** Lives in the
> connection namespace. Your vault uses it to authenticate you to the
> corporate network. Normal handler access, governed by the employment
> contract. If you leave the company, they sever the connection and the
> certificate is no longer accessible to them.
>
> **Employer provisions code-signing key → critical secret.** The
> employer declares this as critical because a compromised code-signing
> key could ship malicious software. Password required for every signing
> operation. The same employer, two different items, two different
> tiers, each appropriate to the risk.
>
> *The connection declares the sensitivity. The vault enforces the
> protection. A retailer's loyalty token gets the second lock --- you
> are asked before it is used. A bank's signing key gets the third lock
> --- your password is required. The architecture applies the right
> protection without you managing it.*

**Three locks, one vault**

The three tiers are not just a classification scheme. They are
enforcement boundaries built into the vault's architecture.

| | **Storage** | **Authentication** | **After use** | **Impact** |
|---|---|---|---|---|
| **Personal data** | Database (personal namespace) | PIN | Unchanged | Recoverable |
| **Secrets** | Database (connection namespace) | PIN + consent gate | Unchanged | Scoped, recoverable |
| **Critical secrets** | Protean Credential | Password + TTL | CEK rotates, old key destroyed | Catastrophic, irreversible |

The first lock --- your PIN --- opens the vault. Everything inside the
database is accessible to the guardian. Personal data and secrets are
available. The guardian serves them through the handler pipeline,
governed by contracts, logged in the audit trail.

The second lock adds consent gating and namespace isolation. Each
connection's secrets are invisible to every other connection, and when a
connection wants to use a secret, the guardian asks you. The PIN opened
the vault. The consent gate ensures your connections do not use your
credentials without your knowledge.

The third lock --- your password --- unlocks the credential. Critical
secrets require this additional authentication, with a user-defined
window for consecutive operations. After every operation, the guardian
rotates the credential's encryption key. The old key is destroyed. The
old credential is worthless. This happens whether you are in the middle
of a TTL window or not --- the protean cycle never pauses, even when the
password challenge does.

The elegance is that you do not manage any of this. You enter your PIN
when you open the app. You enter your password when you use something
critical. The guardian handles the rest: checking contracts, enforcing
tiers, rotating keys, logging access, applying the right lock to the
right data, every time, without exception. The three locks exist so that
security is proportional to risk. Not every piece of data deserves the
heaviest protection. Not every access deserves a password prompt. But
the things that matter most get the strongest lock, and the guardian
never confuses which is which.

> *Three tiers, three locks, one guardian. Personal data gets the PIN
> and the contract. Secrets get the PIN, namespace isolation, and a
> consent gate. Critical secrets get the password, the protean cycle,
> and the user's chosen window. The protection is proportional to the
> risk. The guardian enforces all three. You just use your vault.*
