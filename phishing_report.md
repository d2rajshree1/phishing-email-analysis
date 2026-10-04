# Phishing Email Analysis Report

## 1. Sample Information
- **Source:** phishing_pot collection (https://github.com/rf-peixoto/phishing_pot), file `email/sample-1000.eml`
- **Saved as:** `samples/phish1.eml`
- **Email date:** Wed, 26 Jul 2023 17:59:01 +0000
- **Language:** Brazilian Portuguese
- **Analysis method:** Static, read-only analysis in Kali Linux. No links were visited and the attachment was never opened.

## 2. Sender Analysis
| Field | Value | Observation |
|---|---|---|
| From (display name, decoded) | `[BB] - Seu saldo foi liberado - Código: 11084339647130851921` | Impersonates a bank ("BB" suggests Banco do Brasil); "Your balance has been released" |
| From (address) | `prestonconstance587@gmail.com` | Free webmail account, random name plus digits, not a bank domain |
| Return-Path | `prestonconstance587@gmail.com` | Matches From, but a bank would never use Gmail |
| Reply-To | Not present | - |
| Subject (decoded) | `Liberação de IRPF - 6NwlyfzWcsNerv` | "IRPF release" (Brazilian income tax) plus a random code |

The sender domain (`gmail.com`) is legitimate, so domain age and DNS checks on it were not useful. The attacker used a real throwaway Gmail account and did not spoof the domain.

## 3. Header Analysis
**Authentication results:**
| Check | Result |
|---|---|
| SPF | pass (sender IP 209.85.160.178, a Google server) |
| DKIM | pass (`d=gmail.com`) |
| DMARC | pass (`header.from=gmail.com`) |
| compauth | pass (reason=100) |

All checks pass because the email was genuinely sent through Gmail. Passing authentication shows which domain sent the mail, not that the sender is trustworthy.

**Received chain (origin to delivery):**
1. `from a3.domain ([20.97.213.223]) by smtp.gmail.com with ESMTPSA` (17:59:04 UTC). Authenticated login to Gmail.
2. Gmail internal handoff and relay (17:59:05)
3. Microsoft protection and mailbox servers (17:59:05 to 17:59:07)

**Origin IP findings:**
- `20.97.213.223` belongs to Microsoft Corporation (NetName MSFT, US), which is Azure cloud space.
- No reverse DNS record exists for the IP.
- The HELO name `a3.domain` is not a real domain. It is a made-up hostname typical of bulk-sending scripts.
- Conclusion: the throwaway Gmail account was used from a rented cloud server, not a normal user device.

**Online header analyzer used:** Google Admin Toolbox Messageheader
Screenshot: `evidence/header_analyzer.png`

## 4. Links and Attachments
**Links in the email body (displayed text vs. real URL):** none. There are no HTML anchors and no URLs in the text parts. The attack depends on the attachment.

**Attachment:**
| Field | Value |
|---|---|
| Filename | `csWuYjyqO2IR.pdf` (random name) |
| Type | PDF 1.4, 1 page, A4 |
| SHA-256 | `cfc5fbc759dcc599c8329dd94f9364394c7b1e875d6ff515c7337e00fb1f30cf` |
| Creator / Producer | IronPdf v2022.11.10299 (PDF-generation library) |
| Created | 26 Jul 2023, 09:51 EDT (about 4 hours before the email) |
| JavaScript | No |
| Content | A single 520x735 RGB image and no extractable text. The image was not viewed during analysis, as a precaution. |
| VirusTotal hash search | Not checked |

**Hidden link inside the PDF (mismatched/hidden URL):**
- Defanged URL: `hxxps://okokoaksoa[.]z13[.]web[.]core[.]windows[.]net/`
- A full-page link annotation with a border of 0 makes the whole page an invisible link, so clicking anywhere opens the site.
- The hostname is random. `*.web.core.windows.net` is Microsoft Azure static website hosting, which lets attackers use a trusted domain to avoid reputation filters.
- DNS lookup returned no records, so the page has been taken down (site no longer active at time of analysis).

## 5. Language and Social Engineering
**Translation of the body:**
> Customer [recipient address], Balance for credit to account. More information in the attachment. Protocol: 4793122145117905827 ; 05:59:01 PM

- **Urgent or threatening language:** none found. This email does not rely on fear or deadlines.
- **Financial lure (greed/curiosity):** "balance for credit to account", "IRPF release", "your balance has been released"
- **Brand impersonation:** "[BB]" in the sender name
- **Generic greeting:** "Cliente" instead of a real name
- **Call to action:** "More information in the attachment" pushes the victim to open the PDF
- **Fake legitimacy:** a long "Protocol" number and a timestamp, matching the random code in the sender name
- **Evasion:** very little text and the real content in an image-only PDF, so text-based filters have little to inspect

## 6. Spelling and Grammar Errors
1. "créito" in the HTML part (should be "crédito"), while the plain-text part spells it correctly
2. Greeting merged with the address: `Clientephishing@pot`, and `Clientephishing@pot@hotmail.com` (double @) in the plain-text part, which shows a broken mail-merge template
3. Unescaped HTML entities such as `&eacute;` in the text/plain part
4. Fragment sentences ("Saldo para crédito em conta") and unusually terse wording for a bank
5. 12-hour "05:59:01 PM" timestamp, unusual for Brazilian communication

An automated spell check was not run; the Portuguese text was reviewed manually.

## 7. Summary of Phishing Indicators
| # | Indicator | Evidence | Severity |
|---|---|---|---|
| 1 | Bank impersonation from free webmail | "[BB]" display name, `@gmail.com` address | High |
| 2 | Random sender address | `prestonconstance587` | Medium |
| 3 | Image-only PDF attachment from an unknown sender | `csWuYjyqO2IR.pdf`, no extractable text | High |
| 4 | Hidden full-page link in the PDF | Border-0 URI annotation to an Azure static site | High |
| 5 | Random hostname on trusted cloud hosting | `okokoaksoa.z13.web.core.windows.net` | High |
| 6 | Bogus HELO and cloud origin IP | `a3.domain`, 20.97.213.223 (Azure), no rDNS | Medium |
| 7 | Financial lure | "Saldo para crédito", "Liberação de IRPF" | Medium |
| 8 | Generic, broken greeting | "Cliente" merged with address, double @ | Medium |
| 9 | Fake protocol number | Random number and timestamp | Low |
| 10 | Spelling and formatting errors | "créito", raw HTML entities | Low |
| 11 | Random file name and PDF producer | `csWuYjyqO2IR.pdf`, IronPdf | Low |

Note: SPF, DKIM and DMARC all passed, so passing authentication does not mean an email is safe.

## 8. Conclusion and Recommendations
**Verdict: phishing (credential/financial lure delivered via a malicious PDF link).**

How a user should respond:
- Do not open the attachment or click anything in it.
- Do not reply.
- Report the message as phishing to the mail provider and delete it.
- Banks do not send balance notices from free webmail accounts. Check directly with the bank through its official app or website.
- Mail administrators can block the sender address, the attachment hash and the hosting domain pattern.
