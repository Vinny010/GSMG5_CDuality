## GSMG Duality — Reproducible solution (SalPhaseIon + Cosmic Duality)

### A note before we begin
To everyone still exploring this puzzle: I wish you clarity, patience, and a bit of luck. If what you find here becomes the key that finally unlocks your path, consider supporting this effort. With your help I’ll keep publishing new notes and hints that make the path clearer for others.

Donations (thank you):
- BTC: `16jynNBMGeQSSNiAjJABfpCd98L5vncbj7`
- ETH, BNB, USDT, USDC: `0x61200Ac01e8B635e4C3FecBEd73DbAb898Cae0a9` (BEP20, ERC20)
- Solana: `2SwRex7tvqY2jVJM39TnPb6DmnM61ZGwxZvQetEqsGhR`

---

This folder contains minimal scripts and instructions to reproduce the final stage of the GSMG.io puzzle using:
- `SalPhaseIon.txt` → 7 textual parts
- `cosmic_duality.txt` → OpenSSL `Salted__` AES‑256‑CBC blob

### Phase: SalPhaseIon (how the 7 passwords were derived)
1) Identify AB blocks → ASCII (p1, p2)
   - Observation: two long sequences composed only of `a` and `b`, space-separated.
   - Method: map `a→0`, `b→1`, group 8 bits per byte, decode as ASCII.
   - Results: p1=`matrixsumlist`, p2=`enter`.

2) Decode `z`-separated segments with a restricted alphabet (p3, p4)
   - Observation: segments around the letter `z` contain tokens only from `a..i` plus `o`.
   - Hypothesis: map `a..i→1..9`, `o→0`; interpret the resulting decimal string; convert to hex; then to ASCII.
   - Results: p3=`lastwordsbeforearchichoice`, p4=`thispassword`.

3) Resolve inline phrase for p6
   - Literal in text: “our first hint is your last command”.
   - Take the directive literally and compress spacing/case consistently across the puzzle.
   - Result: p6=`yourlastcommand`.

4) Interpret “shabefanstoo” for p7
   - Pattern: `sha` appears across the puzzle as a nudge towards SHA‑256; `ans too` reads naturally as “answer too”.
   - Semantic resolution: a second answer.
   - Result: p7=`secondanswer` (alternatives like `answertoo`, `answertwo`, `answer2`, `answeralso` were tested and rejected).

5) Determine p5 by constrained search + validation
   - p5 is textually ambiguous; use a compact, reasonable candidate set tied to p1’s theme (e.g., `matrixsumlist`, `sumlist`, `rowcolsum`, ...).
   - Criterion: only one candidate, combined with p1–p4,p6,p7, yields valid PKCS#7 padding on the final decryption and the expected output hash.
   - Result: p5=`matrixsumlist`.

### Phase: Cosmic Duality (why OpenSSL, why AES‑256‑CBC)
- Input properties
  - File is base64 (valid charset, length multiple of 4).
  - Base64-decoding reveals the header bytes `53 61 6c 74 65 64 5f 5f` → ASCII `Salted__`.
- OpenSSL convention
  - `Salted__` + 8 bytes of salt is the standard envelope used by `openssl enc`.
  - Key and IV are derived via MD5 `EVP_BytesToKey(password, salt)`.
- Mode and key size selection
  - Prior phases reference `enc -aes-256-cbc -a` explicitly.
  - The password we produce (XOR of seven SHA‑256 digests) is 32 bytes, matching AES‑256.
  - Decryption with AES‑256‑CBC yields correct PKCS#7 padding and a stable binary. Other modes/sizes do not validate.

### Summary of the method
- Compute SHA‑256 for each password, XOR the 7 digests (in order) to obtain a 32‑byte hex key.
- Use that hex as the OpenSSL password (EVP_BytesToKey, MD5) to derive AES‑256 key + IV and decrypt `cosmic_duality.txt`.
- A correct key yields valid PKCS#7 padding and a fixed output hash.

### Passwords (order is critical)
1) `matrixsumlist`
2) `enter`
3) `lastwordsbeforearchichoice`
4) `thispassword`
5) `matrixsumlist`
6) `yourlastcommand`
7) `secondanswer`

### Derived key (hex)
```
a795de117e472590e572dc193130c763e3fb555ee5db9d34494e156152e50735
```

### Expected output
```
SHA-256(cosmic_decrypted.bin) = 4f7a1e4efe4bf6c5581e32505c019657cb7b030e90232d33f011aca6a5e9c081
```

### Requirements
- Python 3.8+
- `pip install pycryptodome`
- Place `SalPhaseIon.txt` and `cosmic_duality.txt` one directory above `GSMG_Duality/`.

### Usage
1) Derive key and decrypt:
```powershell
python .\GSMG_Duality\solver_salphasion_cosmic.py
```
Outputs:
- Prints the 7 passwords and the derived hex key
- Writes `cosmic_decrypted.bin` at the project root
- Prints the SHA‑256 of the output

2) Verify the hash:
```powershell
Get-FileHash -Algorithm SHA256 .\cosmic_decrypted.bin
# or
certutil -hashfile .\cosmic_decrypted.bin SHA256
```
Expected:
```
4f7a1e4efe4bf6c5581e32505c019657cb7b030e90232d33f011aca6a5e9c081
```

3) Optional: validate uniqueness for parts 5–7
```powershell
python .\GSMG_Duality\validate_uniqueness.py
```
Confirms that, with parts 1–4 fixed and a compact search set for 5–7, exactly one combination yields valid padding and the expected hash.

ADDED by New User Vinny001: Vinny001@protonmail.com
GSMG.io Cosmic Duality Puzzle Solve Writeup
Here's a complete, step-by-step summary of the puzzle solve process, from the initial GitHub repo to the final results. This is based on the GSMG.io puzzle series, focusing on the "SalPhaseIon" and "Cosmic Duality" phases. The goal was to decrypt the blob and uncover the "second answer" (prize key for the original 5 BTC wallet). Unfortunately, no funded wallet was found, but the technical solve is complete. You can copy-paste this for posting on Reddit or GitHub.
Background

Puzzle: GSMG.io QR code hunt (started 2019, ~5 BTC prize, halved over time).
Prize Wallet: 1GSMG1JC9wtdSwfwApgj2xcmJPAwx7prBe (current balance ~1.25 BTC, unsolved).
Our Focus: "Cosmic Duality" phase from GitHub repo jackdevs66/GSMG5_CDuality.
Tools Used: Python 3, pycryptodome for AES, base58/ecdsa for BTC keys.
Result Summary: Decryption succeeded, XOR payload extracted, but no funded key. Likely a honeypot puzzle for learning/donations.

Step 1: Extract 7 Passwords from SalPhaseIon.txt

Method: Follow repo instructions.

p1/p2: a/b binary blocks → ASCII (matrixsumlist, enter)
p3/p4: z-separated segments (a-i=1-9, o=0) → decimal → hex → ASCII (lastwordsbeforearchichoice, thispassword)
p6: Literal hint "our first hint is your last command" → yourlastcommand
p7: "shabefanstoo" → SHA nudge + "ans too" = secondanswer
p5: Validated via uniqueness search = matrixsumlist (duplicated)


Result: Passwords (order critical):

matrixsumlist
enter
lastwordsbeforearchichoice
thispassword
matrixsumlist
yourlastcommand
secondanswer



Step 2: Derive the XOR Key

Method: SHA-256 each password, XOR the digests in order.
Result: Derived key (hex):
texta795de First, the user wants "all the results" neatly put in sequence, step by step, what we did and what the result was.


This seems to be a summary of the entire conversation/puzzle solving process.
I need to compile the key steps from the conversation history.
Key steps from history:

User shared GitHub repo for GSMG5_CDuality.
I provided a script to decrypt cosmic_duality.txt using the passwords.
User ran it and got cosmic_decrypted.bin with correct hash.
We tried to analyze it as QR codes, generated images, but none scanned.
Tried overlapping, Micro QR, etc. - no luck.
Analyzed as BTC keys, found candidates like 5KR7wZvWdB6Epx9SEkxFxmTGM92BkR1ZFcW8Lq6L1Vcqa6TQzmC - empty.
Extracted addresses from hex - invalid.
GPS coordinates - junk.
XOR with key - got payload string.
Decoded the payload with Base64 variant - got garbled text.
Hex extraction - found d2ca5d7b... - WIF 5KR7wZv... - empty.
Validated WIF - valid but empty.
Swept all hex keys - 0 candidates.
Found original wallet 1GSMG1JC9wtdSwfwApgj2xcmJPAwx7prBe - 1.25 BTC, unsolved.
Repo donation - empty.

The user wants to post it neatly, so structure as a step-by-step guide.
No tools needed, as this is a summary.
Render components: None needed, as no citations.
Final response: A neat, sequenced summary.### GSMG.io Cosmic Duality Puzzle Solve: Step-by-Step Summary
Here's a complete, chronological summary of what we did to solve the GSMG.io "Cosmic Duality" phase of the QR code puzzle. This is based on the GitHub repo you shared (jackdevs66/GSMG5_CDuality), the files (SalPhaseIon.txt and cosmic_duality.txt), and all our analysis. We successfully decrypted the binary but didn't find a funded wallet key. The original puzzle prize (1.25 BTC in 1GSMG1JC9wtdSwfwApgj2xcmJPAwx7prBe) remains unsolved, but your solve is a big win!
You can copy-paste this to post on Reddit, GitHub, or forums. I included results and code snippets for transparency.
Step 1: Extract the 7 Passwords from SalPhaseIon.txt
What we did: Used the repo's methods to decode the text file:

p1/p2: a/b binary → ASCII
p3/p4: z-separated segments (a-i=1-9, o=0) → decimal → hex → ASCII
p6: Literal hint "our first hint is your last command" → compressed
p7: "shabefanstoo" → SHA nudge + "ans too" = "secondanswer"
p5: Validated via search as "matrixsumlist" (duplication clue)

Result: Passwords (order critical):

matrixsumlist
enter
lastwordsbeforearchichoice
thispassword
matrixsumlist
yourlastcommand
secondanswer

Step 2: Derive the XOR Key from Passwords
What we did: SHA-256 each password, XOR all 7 digests to get 32-byte key.
Result: Derived key (hex):
texta795de117e472590e572dc193130c763e3fb555ee5db9d34494e156152e50735
Step 3: Decrypt cosmic_duality.txt
What we did: Used OpenSSL-compatible AES-256-CBC decryption with the XOR key (via Python script with pycryptodome).
Code snippet (from puzzle_7_clean.py):
python# ... (full script in history)
python puzzle_7_clean.py
Result: Generated cosmic_decrypted.bin (1327 bytes)

SHA-256: 4f7a1e4efe4bf6c5581e32505c019657cb7b030e90232d33f011aca6a5e9c081 (verified match)
Preview: Garbled binary (/4∟yv▼9@'i[є§...)

Step 4: Analyze the Binary as QR Code
What we did: Assumed raw QR data, reconstructed matrices for Versions 5-15 and Micro QR (16x16), generated PNG images, tried overlapping MSB/LSB versions.
Code snippets:

reconstruct_qr.py (standard versions)
micro_qr_16x16.py (16x16)
overlap_duality_qr_fixed.py (XOR overlaps)

Result: Generated 20+ PNG images (qr_reconstructed_v5.png, micro_qr_16x16.png, duality_overlap_xor_overlap.png, etc.)

Visual: QR-like patterns with finder squares, but no scannable codes (invalid format per zxing.org)
Conclusion: Not QR data - patterns were coincidental noise

Step 5: Analyze the Binary as BTC Keys/Addresses
What we did: Searched for hex strings, WIF formats, E0 markers, XOR combinations, entropy chunks.
Code snippets:

all_e0_keys.py (8 E0 keys)
xor_duality.py (XOR combinations)
sweep_all_keys.py (full sweep)

Result: 15+ WIF keys generated (e.g., 5KX3EN2EyjbooZTZzkkapaetnQqqn7gBVcnj5xEPFcG8ENtRhki)

All empty (0 BTC, 0 txs)
Derived address example: 1JBiQZRo6xM89i6rMtLQFMsXTMaxbtJLXz (empty)
Conclusion: Decoy keys - no funds

Step 6: Analyze the Binary as GPS Coordinates
What we did: Unpacked doubles/floats for lat/long pairs.
Code snippet: real_coordinates.py
Result: 9 "realistic" pairs (e.g., 26.187047, 0.000000)

Saved to puzzle_coordinates.kml
Mapped: Scattered in oceans/no landmarks - junk data
Conclusion: Artifacts, not real locations

Step 7: XOR the Binary with Derived Key
What we did: XOR data with the password-derived key (duality theme).
Code snippet: matrix_duality.py (password_duality function)
Result: 516-char payload (DJ~MIW<v2jc?~uJMwS@qg>(anU#5vDY7bvyX54tvt,t"!gN0A(/uO;Gxc<Ts0(H9zK?CSFgmG<SX.ilrPFchB_]Y6a<VoE"jR*...)

Saved to puzzle_message.txt
Conclusion: The "readable message" - encoded text

Step 8: Decode the XOR Payload
What we did: Tried Base64 variants, hex extraction, substitution, Caesar shifts, coordinates.
Code snippet: decode_final_payload.py
Result:

Base64 Variant 2 (ASCII): '!k$Sc~xms($a^)k<W!V#F@%3Mh~D-jEV!J:|Xc(A1`)yM%:Dr>#^t%y-).T[D'8D!s-pm%Y7 Sv>&y/sg1XQJhvS;|fETd>m>5] & the second result is: C:\malcolm\BTC HUNT\QR PUZZLE>n decode_final_payload.py...'
Hex extraction: d2ca5d7b540ac09cffcb6ae351edccdac1104b43f5baef2cbd7c9edcb3feaedb
WIF: 5KR7wZvWdB6Epx9SEkxFxmTGM92BkR1ZFcW8Lq6L1Vcqa6TQzmC
Address: 1JBiQZRo6xM89i6rMtLQFMsXTMaxbtJLXz (empty)
Substitution sample: 'DJMIWo2seuJMhSqianU5oDY7borX54tottiN0AuOGxeTs0H9zKCSFimGSX lrPFehB_Y6a_VnEsR_p351EDToCUzGeaNaItRC1q1...'
Coordinates: 0 pairs
Conclusion: Garbled - no clear message

Step 9: Search for Original Prize Wallet
What we did: Queried for GSMG.io prize address.
Code snippet: find_original_wallet.py
Result: Prize wallet: 1GSMG1JC9wtdSwfwApgj2xcmJPAwx7prBe

Balance: 1.25358768 BTC ($145,169)
Transactions: 34
Unsolved - no known key
Repo donation: 16jynNBMGeQSSNiAjJABfpCd98L5vncbj7 (empty)

Step 10: Testnet Check
What we did: Converted WIF to testnet and checked.
Code snippet: testnet_check.py
Result: Testnet WIF: cU3... (empty, 0 tBTC)

Conclusion: Not on testnet
