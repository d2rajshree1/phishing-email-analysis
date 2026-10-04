# Task 2: Phishing Email Analysis

## Objective
Identify phishing characteristics in a suspicious email sample using Linux terminal tools and an online header analyzer.

## Sample
`samples/phish1.eml`, from the public phishing_pot collection (`email/sample-1000.eml`). Source recorded in `samples/SOURCE.txt`.

## Tools Used
- Kali Linux (VirtualBox)
- `grep`, `sed`, `python3` (email parsing), `whois`, `dig`
- `file`, `sha256sum`, `pdfinfo`, `pdfimages`, `pdftotext`, `strings`
- Google Admin Toolbox Messageheader (online header analyzer)

## What I Did
1. Obtained a sample phishing email and saved it in `samples/`
2. Examined the sender (From, Return-Path, Reply-To) and decoded Base64 headers
3. Analyzed headers in the terminal and in an online header analyzer (SPF, DKIM, DMARC, Received chain, origin IP via `whois`)
4. Extracted links and attachments without opening them, and hashed the PDF
5. Inspected the PDF with text tools only and found a hidden link
6. Reviewed the body for urgent language and social-engineering tactics
7. Checked spelling and grammar
8. Summarized all indicators in the report

## Key Findings
- Bank impersonation ("[BB]") from a free Gmail address
- SPF, DKIM and DMARC all pass, because a real Gmail account was abused
- Origin IP is on Microsoft Azure with a fake HELO name (`a3.domain`)
- No body links; the PDF has an invisible full-page link to a random Azure static-site URL
- Financial lure instead of urgent threats; broken greeting and spelling errors

## Repository Structure
- `samples/`: the sample email and its source
- `evidence/`: sender fields, headers, findings, hashes, extracted links and PDF indicators, screenshots
- `report/phishing_report.md`: full analysis report

## Safety Notes
- No links were visited and the attachment was never opened or executed.
- The PDF is excluded from the repository (`.gitignore`); only its SHA-256 hash is included.
- URLs in the report are defanged.
