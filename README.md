# Hacking Notes

Notes distilled from ["The Hacking Iceberg is INSANE"](https://www.youtube.com/watch?v=OmEqWHyXD6Y)
(YouTube, creator `Manware`) — a tour through famous real-world attacks, from
a teenager cold-calling CIA support up to physics-level and biometric
side-channel attacks against fully air-gapped hardware. Unlike the OPSEC
iceberg notes, this video is narrative/historical rather than tool-driven, so
these notes pull the **defensive lesson** out of each case rather than
listing software to install.

## Social engineering beats code

- **2015 CIA director hack** — a 15-year-old got the CIA director's phone
  number, account PIN, email, and password purely by cold-calling customer
  support and posing as an internal technician, answering security questions
  correctly. No exploit code at all.
- **Takeaway**: the weakest link is almost always a human on a support line
  being talked past identity verification, not a vulnerability in software.
  Treat "security questions" (mother's maiden name, pet name, etc.) as
  effectively public information if it can be social-engineered out of a
  support rep.

## Supply-chain trust is a multi-year attack surface

- **XZ Utils backdoor (2021–2024)** — "Jia Tan" spent ~3 years building
  legitimate trust as a contributor/co-maintainer on a single-maintainer,
  unpaid, burned-out open-source compression library used by nearly every
  major Linux distro, then slipped a remote-root SSH-auth-bypass backdoor
  into obfuscated binary test files inside the release tarball (never visible
  in the actual git history/diffs). Caught only because one developer
  (Andres Freund) noticed SSH logins were ~0.5s slower than expected and
  investigated.
- **Takeaway**: a maintainer's burnout + a "helpful" long-term contributor
  pushing for commit access is a real, slow-moving attack pattern — not just
  a theoretical supply-chain-security talking point. Worth being suspicious
  of unusual performance regressions (the thing that actually caught this),
  and of reviewing *release tarballs*, not just the git diff, for critical
  infrastructure dependencies.

## Default credentials and exposed services are still the easy win

- **Mirai botnet (2016)** — exploited the fact that IoT devices (routers,
  CCTV, thermostats) ship with default credentials nobody changes; the
  resulting botnet knocked Dyn (DNS for Netflix/Twitter/Reddit/etc.) offline
  in the largest DDoS at the time. Built by a college student, not a
  nation-state.
- **"Gay Femboy" botnet (modern Mirai descendant)** — no longer brute-forces
  passwords; instead chains known CVEs/n-days/zero-days across many CPU
  architectures, and specifically scans for **unauthenticated, publicly
  exposed Redis instances**. Also fights/kills competing botnets on infected
  hosts.
- **Takeaway, concretely actionable**: change default credentials on every
  IoT device (router, camera, thermostat) on your network, and never expose
  Redis (or any datastore) without authentication on a public interface —
  this is still being actively scanned for in 2024+, not a solved problem.

## Zero-click mobile exploits are real and don't require user error

- **Pegasus / ForcedEntry (2021, against iMessage)** — a JBIG2 image
  compression stream (no scripting capability by design) was abused to build
  an emulated computer (registers, adder, comparator, memory search) purely
  out of primitive bit operations, triggering an integer overflow in Apple's
  Core Graphics and escaping the `BlastDoor` sandbox — all from receiving a
  single message, no tap required.
- **Takeaway**: "don't click suspicious links" doesn't help against zero-click
  attacks. The only real mitigations are keeping the OS fully patched and,
  for high-risk individuals specifically (journalists, activists, dissidents),
  iOS **Lockdown Mode**, which disables exactly this class of complex
  attachment-parsing attack surface.

## Hardware-level attacks that no software patch fixes

- **Rowhammer / NetHammer** — repeatedly accessing the same DRAM row at high
  frequency leaks charge into adjacent rows and flips bits you shouldn't be
  able to touch; used by Google Project Zero (2015) for privilege escalation
  via page-table-entry bit flips, and later shown triggerable remotely over
  crafted network traffic (NetHammer, 2018). No software patch fully closes
  this — it's a physics problem in the DRAM itself.
- **Takeaway**: if you need a real defense here, it's hardware — ECC RAM and
  memory controllers with Target Row Refresh (TRR), not a software update.

## Air-gapped hardware is not actually isolated

Every one of these has real academic names, not just theoretical:

- **AirHopper** — a GPU/display's normal electromagnetic (FM-band) emissions
  can be modulated to exfiltrate data, up to ~7m.
  **Fansmitter** — same idea via controlled fan-noise pitch/rhythm, ~8m.
- **BitWhisper** — CPU-load-driven heat patterns picked up by infrared
  sensors, up to ~1.5km.
- **Jetson/cardiac side-channel (MIT, 2019)** — infrared laser reads
  heartbeat-driven skin micro-vibrations through clothing at ~200m; a
  biometric signature that can't be changed the way a face or gait can.
- **Takeaway**: "air-gapped" only means "no network cable" — it does not mean
  "no side channel." Real air-gap security practice (TEMPEST shielding,
  Faraday enclosures, physical distance control) exists precisely because of
  attacks like these; a laptop sitting on a desk with Wi-Fi disabled is not
  equivalent to a properly shielded facility.
