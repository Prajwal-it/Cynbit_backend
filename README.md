Cynbit Technologies: Backend Technical Audit
Candidate: Prajwal Banthiya | Role: Backend Developer Intern | Target: https://www.cynbit.in
Method: passive only. Public pages read normally. No exploitation, fuzzing, load, authentication attempts or access to non-public systems.
Each item is tagged [Observed] (visible in public page content/metadata) or [Verify] (a hypothesis from the observed architecture, with a passive check to confirm it).
1. Surface map [Observed]
Area
Pages
Backend implication
Marketing
/, /story, /team, /services, /impact, /case-studies, /blog
Cacheable content layer, SEO-driven
Lead capture
/contact (Full Name, Email, Company, Industry, Message)
Public write endpoint: validation, spam control, email delivery
Newsletter
Footer form, "25k+ subscribers" claim
Public write endpoint, opt-in and storage
Education
/bitelevate, /bitelevate/paths, /diaries
Enrollment enquiries, user-generated content
Careers
/careers
Likely resume/application intake
Products
Cynbit Studio, Revio, TenantEase
Likely separate apps with own auth/APIs
Legal
/privacy, /terms, /cookies
Data-handling duties (India DPDP Act 2023)
Note: /privacy, /careers and other sub-pages could not be read in this review, so items about them are [Verify].
2. Architecture signals
Next.js frontend [Observed, inferred]: page metadata contains next-size-adjust, emitted by Next.js. Route handlers under /api/* are the likely backend layer.
Two hostnames in use [Observed]: the shared link resolves to https://www.cynbit.in, but og:url and og:image point to apex https://cynbit.in.
Contact details published in plain text [Observed]: opportunities@cynbit.in, support@cynbit.in and three phone numbers on every page.
Contact form [Observed]: fields Full Name, Email, Company (optional), Industry (optional), Message. The page promises a 24-hour response. No CAPTCHA widget is visible in the rendered content.
Marketing metrics (25k+ subscribers, 1000+ certified, 99.9% uptime) [Observed]: shown as static copy and not verifiable from outside.
Probable API shape [Verify]
Submit the contact form once with test data and record the details in findings/verification-log.md: endpoint, content type, payload fields, error shape and any CAPTCHA token.
3. Security & performance findings
Security
ID
Finding
Status
Severity
S-1
Public write endpoints (contact, newsletter) may lack rate limiting / bot protection
Verify
Medium
S-2
Server-side validation and email header injection risk on free-text fields
Verify
Medium
S-3
Security headers (HSTS, CSP, nosniff, Referrer-Policy, frame protection)
Verify
Low-Medium
S-4
Inconsistent canonical host (www vs apex) affects cookies, CORS, cache keys
Observed
Low
S-5
X-Powered-By / Server framework leakage
Verify
Low
S-6
CORS policy on API routes (wildcard or reflected origins)
Verify
Medium if misconfigured
S-7
Careers/enrollment intake: file type, size and storage handling if resumes accepted
Verify
Medium-High if present
S-8
Newsletter single opt-in and subscriber enumeration via differing responses
Verify
Low-Medium
S-9
Retention, processors and deletion channel under DPDP Act
Verify
Low
S-10
Plain-text emails and phone numbers enable scraping
Observed
Info
Performance
ID
Finding
Status
Severity
P-1
Caching of statically renderable pages (Cache-Control, ISR/SSG vs SSR)
Verify
Medium
P-2
Animated marquee/decorative sections affecting LCP/TBT
Verify
Low-Medium
P-3
Assets served from apex while pages run on www (extra DNS/TLS)
Observed
Low
P-4
Brotli/gzip and HTTP/2-3 support
Verify
Low
P-5
Third-party scripts or icon fonts blocking render
Verify
Low-Medium
Data validation / error handling [Verify]
D-1: inconsistent error contract (HTML 500 vs JSON) may leak stack traces.
D-2: duplicate submissions without idempotency or button disabling.
D-3: client-only required validation without server schema checks.
Authentication
No login surface is visible on the main site. The three SaaS products live behind /products anchors, so their auth is out of passive scope. Recommendation: keep them on separate subdomains with host-only cookies.
4. Recommended improvements
Example code is in snippets/:
Canonical host and security headers: snippets/next.config.js
Validated, rate-limited contact endpoint (zod schema, honeypot, Turnstile, queueing): snippets/contact-route.ts
Consistent error contract: snippets/errors.ts
Newsletter double opt-in without enumeration: snippets/newsletter-flow.md
Performance: static rendering with ISR and Cache-Control: public, s-maxage=3600, stale-while-revalidate=86400 behind a CDN; one canonical asset host with immutable hashed filenames; lazy-load marquee/testimonial sections and respect prefers-reduced-motion; Lighthouse CI to prevent regressions.
Data protection: store inquiries in a database with a retention policy, add a deletion channel to the privacy policy (DPDP Act), keep SaaS products on separate subdomains with __Host- cookies (Secure; HttpOnly; SameSite=Lax), and add structured logging with request IDs and alerts on form endpoints.
5. Scope & ethics
Only public pages and normal browser behaviour were used. No scanning, brute-forcing, injection attempts, load generation or access to non-public data. Items marked [Verify] are hypotheses to confirm with the passive checks in findings/verification-log.md, not claims of confirmed vulnerabilities.
