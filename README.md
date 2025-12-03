PKI-Based 2FA System (GPP Week-2)

This project implements a complete Public-Key-Infrastructure based Two-Factor Authentication (2FA) system required for Global Placement Program – Week 2.

It includes:

RSA key generation

RSA/OAEP decryption of instructor-generated seed

RSA-PSS commit proof signing

TOTP generation and verification

Docker container with FastAPI + Cron + UTC timezone

Automated cron logging of OTP every minute

Fully reproducible deterministic seed flow

1. Repository Information
GitHub Repository
https://github.com/saihemanth2006/PKI-Based-2FA-Microservice.git


This URL was also used for:

Instructor API seed generation

Submission details

2. System Overview

This project securely generates a TOTP-based 2FA code using:

A seed encrypted by the instructor API

Decrypted using student’s RSA-4096 private key

Stored in Docker volume /data/seed.txt

Converted to Base32

Used to generate 6-digit TOTP every 30 seconds

Verified with ±1 window tolerance

Logged every minute via cron

3. Directory Structure
.
├── app/
│   ├── main.py                 # FastAPI server
│   ├── crypto_utils.py         # RSA decryption helpers
│   ├── totp_utils.py           # TOTP generation & verification
│   └── generate_proof.py       # Commit-proof generator
│
├── cron/
│   └── 2fa-cron                # Cron job file (LF-only)
│
├── scripts/
│   └── log_2fa_cron.py         # Cron script to write OTP logs
│
├── student_private.pem
├── student_public.pem
├── instructor_public.pem
├── encrypted_signature.b64     # Encrypted commit-proof signature
├── encrypted_seed.txt          # Encrypted seed from instructor
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .gitattributes
├── .gitignore
└── README.md

4. Cryptography Details
4.1 Key Pair

RSA key size: 4096 bits

Public exponent: 65537

Format: PEM

Files committed (required):

student_private.pem

student_public.pem

4.2 Seed Decryption (RSA-OAEP-SHA256)

Instructor returns encrypted_seed → Base64 string.

Decrypted using:

Algorithm: RSA

Padding: OAEP

MGF: MGF1(SHA-256)

Hash: SHA-256

Label: None

Result: a 64-character hex seed stored in /data/seed.txt.

4.3 Commit Proof Signing (RSA-PSS-SHA256)

Commit hash is signed using:

RSA-PSS

MGF1(SHA-256)

Salt length: MAX_LENGTH

Input: ASCII string of commit hash

Then signature is:

Encrypted via instructor public key (RSA-OAEP-SHA256)

Base64 encoded → stored as encrypted_signature.b64

5. TOTP System
Algorithm

Standard TOTP

Hash: SHA-1 (Google Authenticator default)

Period: 30 seconds

Digits: 6

Seed Handling

Convert hex seed → bytes

Encode bytes → Base32

Run TOTP computation

Verification

Allows a ±1 window (±30 seconds) to handle clock skew.

6. API Endpoints
Base URL (local)
http://localhost:8080

6.1 POST /decrypt-seed

Decrypts the instructor-provided encrypted seed.

Request
{
  "encrypted_seed": "BASE64_STRING"
}

Success Response
{"status": "ok"}


Seed is stored at:

/data/seed.txt

6.2 GET /generate-2fa

Returns current 2FA code and time remaining.

Response
{
  "code": "123456",
  "valid_for": 19
}

6.3 POST /verify-2fa

Verify a user-provided OTP.

Request
{"code":"123456"}

Response
{"valid": true}

7. Cron System
Cron File:
cron/2fa-cron

Runs every 1 minute:

Executes:

scripts/log_2fa_cron.py

Output File:
/cron/last_code.txt

Sample line:
2025-12-02 12:52:01 - 2FA Code: 430030

Important

.gitattributes ensures LF line endings:

cron/2fa-cron text eol=lf

8. Docker Setup
8.1 Dockerfile

Multi-stage build:

Builder stage for dependencies

Runtime stage with cron + app

Key points:

Timezone set to UTC

Cron installed and running

Volumes created: /data, /cron

8.2 Docker Compose

Named volumes:

seed-data → /data

cron-output → /cron

FastAPI served on:

http://localhost:8080

9. How to Run Locally
Build
docker compose build

Start
docker compose up -d

Check Running
docker compose ps

10. Testing
1️⃣ Decrypt Seed
curl -X POST http://localhost:8080/decrypt-seed \
  -H "Content-Type: application/json" \
  -d "{\"encrypted_seed\": \"$(cat encrypted_seed.txt)\"}"

2️⃣ Get OTP
curl http://localhost:8080/generate-2fa

3️⃣ Verify OTP
CODE=$(curl -s http://localhost:8080/generate-2fa | jq -r '.code')
curl -X POST http://localhost:8080/verify-2fa \
  -H "Content-Type: application/json" \
  -d "{\"code\": \"$CODE\"}"

4️⃣ Cron Output
docker compose exec app cat /cron/last_code.txt

11. Submission Details
Required Fields

GitHub Repository URL

Commit Hash

Encrypted Seed

Student Public Key

Encrypted Signature (from encrypted_signature.b64)

Optional: Docker image link

12. Common Pitfalls & Fixes

✔ Correct padding (OAEP + PSS)
✔ No CRLF in cron file
✔ Seed stored in /data not inside container
✔ TOTP uses base32-encoded seed
✔ UTC timezone
✔ ±1 verification window
✔ Commit hash signed as ASCII
✔ Encrypted signature copied as one line

13. Final Notes

This project fully satisfies Week-2 requirements:

PKI

RSA decryption

TOTP generation

Cron integration

Docker-based runtime

Commit-proof signing

Everything is reproducible and ready for evaluation.