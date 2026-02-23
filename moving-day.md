**Moving Day**

*How you move your vault without losing your vine*

February 2026

**The promise**

ZKT says you own your digital identity. That claim is meaningless if you
cannot leave.

If your vault provider is the only place your secrets can live, you do
not own your digital identity. The provider owns it and you rent it. If
the provider raises prices, degrades service, gets acquired, shuts down,
or simply builds a product you no longer want, you should be able to
take your data and move to a different provider. Not export a zip file
and hope the next provider can read it. Move. Vault to vault. Your
connections stay intact. Your contracts carry forward. Your vine keeps
growing from a new root.

This is not a feature VettID has built today. It is a capability the
architecture must support, and the design principles are clear enough to
lay out now. Portability is the structural guarantee that the vine
belongs to you, not to the provider who happens to operate your tower.

> *You own your digital identity only if you can leave your vault
> provider and take everything with you. Portability is not a feature.
> It is the proof that the ownership is real.*

**Two principles**

Before getting into how a transfer works, two principles constrain the
design.

**The provider enables it**

Portability is a capability that vault providers offer to their users.
It is not something the user has to engineer on their own. The vault
provider is responsible for implementing the transfer protocol,
supporting inbound and outbound transfers, and ensuring that the process
works reliably. A vault provider that does not support portability is
making a lock-in decision, and users should know that before they
enroll.

This is similar to how phone number portability works. The carriers
built the infrastructure. You initiate the transfer. The system handles
the rest. You should not need a computer science degree to move your
vault.

**No new complexity or risk for the user**

The transfer cannot expose secrets to the user's device, to the network,
or to any system outside a hardware-isolated enclave. There is no export
file. There is no "download your data" step where a JSON file full of
decrypted credentials sits on your phone or in a cloud bucket. The data
moves enclave to enclave, encrypted in transit, through the messaging
layer. The user initiates the transfer, confirms a few steps, and waits.
The complexity is in the architecture, not in the user's hands.

> *The provider enables portability. The user initiates it. Secrets move
> enclave to enclave, never exposed. No export files. No decrypted data
> on devices. The complexity lives in the architecture, not in the
> user's experience.*

**The move**

Here is how a transfer works, step by step. You have a vault with
Provider A. You want to move to Provider B.

**Step 1: Enroll with Provider B**

You sign up with Provider B and create a new vault. You go through the
normal enrollment: create a PIN, create a password, establish your
Protean Credential. At this point, Vault B exists but is empty and
inactive. It is set up to receive an inbound transfer and acknowledge
connections, but it will not do anything else until the transfer is
complete. It is a new tower, built and waiting, with the guardian
standing inside an empty room.

**Step 2: Connect Vault A to Vault B**

You create a connection between your two vaults. This is a
vault-to-vault connection through the messaging layer, the same kind of
connection that exists between any two vaults on the vine. Vault A and
Vault B establish end-to-end encryption keys and can communicate
securely.

**Step 3: Initiate transfer from Vault A**

You tell Vault A: transfer everything to Vault B. At this moment, Vault
A freezes. It stops accepting new writes to its database. No new data is
created, no state changes, no updates. The database is static. This
freeze is critical --- it means the data being transferred is a clean,
consistent snapshot with nothing changing underneath it.

Any requests that arrive during the transfer --- a service making a data
request, a friend sending a message, a contract update notification ---
go into a queue. Vault A is not dropping events. It is collecting them
in order, holding them for later.

**Step 4: Vault B accepts**

Vault B confirms it is ready to receive. The two vault managers
establish the transfer channel through the messaging layer, encrypted
end-to-end, attested on both sides.

**Step 5: Data transfer**

Vault A sends the vault data to Vault B through the messaging layer.
Enclave to enclave. The data is encrypted in transit and only
decryptable inside Vault B's enclave. This includes the personal
namespace, all connection namespaces with their keychain rings, all
service namespaces with their stored data, all shared sandboxes the user
hosts, and all active connection contracts.

Contracts transfer as-is. A contract is between you and the connection,
not between you and your vault provider. Vault B inherits every contract
from Vault A and enforces them going forward. The governance of your
relationships does not change because you changed providers.

**Step 6: Critical secrets**

Critical secrets live inside the Protean Credential, not in the
database. Vault A's vault manager extracts your critical secrets from
Vault A's credential, encrypts them, and sends them to Vault B through
the secure channel. Vault B's vault manager receives them and has you
add them to your new Vault B credential. You are involved in this step
--- your password is required to authorize the addition of each critical
secret into the new credential. This is consistent with the model:
critical secrets require your active authentication.

**Step 7: Connection update**

Once the data and critical secrets are in Vault B, the connections need
to know where to find you. Vault B generates new messaging details for
each existing connection --- the new MessageSpace address and access
credentials. These details are sent back to Vault A, and Vault A
distributes them to every connection: friends, services, employers,
every vault on your vine.

Each connection acknowledges the update and verifies that it can reach
Vault B. This is the handshake: the connection receives the new address,
connects to Vault B's MessageSpace, confirms the channel is live, and
acknowledges. This should take minutes. Your root should be clean ---
nothing dead, nothing stale, nothing that has been inactive for months.
If your vine is well-maintained, every connection acknowledges quickly.

Shared sandboxes are updated automatically as part of this step. If you
host a shared sandbox, the connection associated with that sandbox
receives the updated address along with every other connection. The
sandbox data already transferred in Step 5. The connection just needs to
know the new location.

**Step 8: Queue delivery and completion**

After all connections have acknowledged, Vault A delivers the queue ---
every event that arrived during the transfer, in order --- to Vault B.
This is one of Vault A's final acts. Vault B processes the queue as its
first real work: handling the service requests, delivering the messages,
processing the contract updates. Nothing was lost. Nothing was out of
order. The queue bridges the gap.

Vault A's vault manager terminates. Vault A may be deleted. You are now
operating entirely from Vault B. Your vine is intact. Your connections
reach you at the new address. Your contracts are enforced by the new
guardian. Your critical secrets live in the new credential. Moving day
is over.

> *Freeze, transfer, update, queue, complete. Vault A stops writing.
> Data moves enclave to enclave. Critical secrets go credential to
> credential. Connections get new addresses. The queue bridges the gap.
> Vault B takes over. The vine never breaks.*

**What moves and what stays**

Not everything goes to the new vault. The transfer is deliberate about
what crosses over and what remains behind.

**What moves**

| **Data** | **How it transfers** |
|---|---|
| **Personal namespace** | Your profile fields, personal settings, identity data. Sent through messaging layer, enclave to enclave. |
| **Connection namespaces** | Every connection's keychain rings, secrets, stored data. Each namespace transfers intact. |
| **Service namespaces** | Service-stored data (opaque rings), service profiles, interaction history. |
| **Shared sandboxes** | All sandboxes the user hosts. Associated connections are updated automatically. |
| **Connection contracts** | Every active contract, as-is. Vault B inherits and enforces them going forward. |
| **Critical secrets** | Extracted from Vault A's credential, sent encrypted, added to Vault B's credential with user authentication. |

**What stays**

Audit logs do not transfer to the new vault. The audit trail is a record
of what happened under Vault A's watch, and it stays with Vault A. Vault
B starts a fresh audit trail from the moment it begins processing
events. This is clean separation: the old record belongs to the old
guardian. The new guardian writes its own.

However, audit logs are not trapped. They can be exported from Vault A
before or after the transfer. If you want a complete archive, you export
the logs and store them wherever you choose. And vault providers may
offer log forwarding --- a connection to a SIEM, a compliance platform,
or any external system --- so that your vault can send all or some of
the audit trail to other systems in real time.

Log forwarding is governed by a connection contract, the same as any
other connection. The contract declares what log data gets forwarded, to
where, at what frequency, and with what retention. This is the same
consent model that governs everything else on the vine. Your audit data
goes where you say it goes, under terms you accepted, enforced by the
vault.

> *Vault data moves. Audit logs stay. Logs can be exported or forwarded
> under a connection contract. The old guardian's record stays with the
> old guardian. The new guardian starts fresh. Your data follows you.
> Your history is available but separate.*

**Rearranging furniture**

Provider B may organize things differently than Provider A. Different
handlers mean different capabilities. A feature that Provider A offered
as a built-in handler might not exist in Provider B's handler set, or it
might work differently.

Your connections are automatically updated with your new profile from
Vault B. The handler manifests in your profile tell connections what
your vault can now do. If Provider B supports all the same handlers,
nothing changes from your connections' perspective. If Provider B has
different capabilities, connections see the updated manifest and adjust
accordingly.

On your side, you may need to organize your data to align with Provider
B's features. This is like moving to a new apartment: the furniture is
the same, but the rooms are shaped differently. Your credentials, your
secrets, your connections --- all of it transferred. But how Provider B
presents, manages, and extends that data may be different from what you
are used to. This is rearranging furniture, not replacing it.

This is also where you experience the vault provider's product. Provider
A might have been strong in financial handlers but weak in healthcare.
Provider B might be the opposite. The data is the same. The capabilities
are different. You chose Provider B for a reason, and the rearranging is
the process of settling into the new tower.

> *The data transfers. The capabilities may differ. Your connections see
> the new profile and adjust. You may need to rearrange how things are
> organized --- same furniture, different rooms. That is the cost of
> switching. The vine stays intact.*

**Why this matters**

Portability is the mechanism that keeps vault providers honest.

If you cannot leave, the provider has leverage. They can raise prices
because you have no alternative. They can degrade service because
switching costs are too high. They can change terms because your data is
trapped. Every platform you have ever used that made leaving difficult
was using that difficulty as a business model. Your data was the moat.

ZKT portability eliminates the moat. Your data moves vault to vault
through the messaging layer. Your connections update automatically. Your
contracts carry forward. Your critical secrets transfer credential to
credential. The process takes minutes, not months. And because the
transfer runs through the same infrastructure that handles all vault
communication --- the same messaging layer, the same encryption, the
same attestation --- it inherits all the same security properties.
Secrets are never exposed. The transfer is logged. Both enclaves are
attested.

This changes the competitive dynamics for vault providers. Providers
cannot compete on lock-in. They have to compete on the product: better
handlers, better user experience, better reliability, better
specialization. If your provider is not serving you well, you move. The
vine follows. The provider is left with an empty tower.

**The standard makes it real**

Portability depends on the messaging standard described in The Nervous
System. If every vault provider speaks the same protocol --- the same
event format, the same envelope encryption, the same topic conventions,
the same connection handshake --- then a transfer from Provider A to
Provider B is just vault-to-vault communication using the same
infrastructure that handles every other interaction on the vine. The
transfer protocol is an extension of the messaging standard, not a
separate system.

Without the standard, portability is a promise each provider may or may
not honor. With the standard, it is a structural property of the
ecosystem. Any two compliant vaults can transfer data between them
because they speak the same language. The user does not need to check
whether Provider A and Provider B are "compatible." If both are on the
vine, the transfer works.

This is the final piece of the ownership claim. Your data is in your
vault. Your vault is in an enclave you can attest. Your contracts are
enforced by your guardian. Your audit trail is in your hands. And if you
want to change providers, you connect two vaults, initiate a transfer,
and your vine keeps growing from a new root. The provider operates the
tower. You own everything inside it. And you can prove it by leaving.

> *Portability is not a feature. It is the proof that ownership is real.
> If you can leave and take everything with you --- data, secrets,
> connections, contracts --- then the vault provider is a service, not a
> landlord. The vine belongs to you. Moving day proves it.*
