# Username Reserved & Blocked Lists V1

**Status:** Approved

This document defines every username that is unavailable for public registration
in Minime V1, across three independent lists:

1. **System Reserved** — protects platform routes and internal functionality.
2. **Brand & Identity Reserved** — protects brands, public figures, organizations, and strategic assets.
3. **Blocked Terms** — prohibits offensive, abusive, or unsafe terms.

---

## Shared Rules

These rules apply to all three lists.

- **Result.** Any username matching any list returns `blocked` during username
  availability checks.
- **Matching.** Detection uses **substring matching**. Example: the term `admin`
  blocks `admin`, `admin1`, `myadmin`, `adminpanel`, `superadmin`.
- **Management.** All lists are maintained internally by Minime. Users cannot
  claim, request, appeal, or override entries. Ownership verification is not
  performed in V1.
- **User visibility.** The system returns only `blocked`; it never reveals which
  list or term caused the rejection.
- **Extensibility.** Terms may be added or removed without modifying the Account
  Claim System. Only this document changes.

---

## 1. System Reserved

Protects platform routes, internal features, administrative areas, infrastructure
endpoints, and future platform capabilities.

**Authentication & Access:** `login` `signin` `signup` `register` `logout` `auth` `authentication` `verify` `verification` `otp`

**Account Management:** `account` `accounts` `profile` `profiles` `settings` `security` `billing` `subscription` `subscriptions` `plan` `plans` `upgrade`

**Administrative Functions:** `admin` `administrator` `moderator` `staff` `team` `owner` `management` `console` `controlpanel`

**Platform Infrastructure:** `api` `cdn` `assets` `static` `media` `storage` `files` `upload` `uploads` `download` `downloads`

**System Operations:** `system` `internal` `private` `public` `service` `services` `status` `health` `monitor` `monitoring` `logs`

**Website Navigation:** `home` `about` `contact` `help` `support` `faq` `docs` `documentation` `blog` `news` `careers` `jobs`

**Legal & Compliance:** `terms` `privacy` `cookies` `policy` `policies` `legal` `compliance` `gdpr`

**Social Accounts:** `discovery` `discover` `search` `scanner` `scan` `finder` `import` `importer`

**Analytics:** `analytics` `stats` `statistics` `reports` `reporting` `insights` `metrics` `tracking`

**Platform Branding:** `minime` `minimeae` `official` `verified` `premium` `pro` `enterprise` `business`

Future reserved categories (AI features, commerce, payments, creator programs,
advertising, partner systems, mobile apps, developer tools) may be expanded later.

---

## 2. Brand & Identity Reserved

Protects global brands, companies, public figures, organizations, governments,
countries, and high-value strategic assets. The initial list is curated and is
expected to grow over time.

**Global Brands:** `nike` `adidas` `apple` `google` `microsoft` `amazon` `meta` `facebook` `instagram` `threads` `tiktok` `youtube` `spotify` `netflix` `tesla` `openai` `chatgpt`

**Major Technology Companies:** `samsung` `intel` `amd` `nvidia` `oracle` `ibm` `cisco` `paypal` `stripe` `shopify`

**Global Public Figures:** `messi` `cristiano` `ronaldo` `neymar` `mbappe`

**International Celebrities:** `taylorswift` `kimkardashian` `therock` `mrbeast`

**Arab Celebrities:** `ahmedelsakka` `mohamedramadan` `amrdiab` `tamerhosny` `adelimam`

**Sports Organizations:** `fifa` `uefa` `laliga` `premierleague` `nba`

**News Organizations:** `bbc` `cnn` `reuters` `aljazeera` `skynews`

**Government Entities:** `whitehouse` `pentagon` `nasa` `unesco` `unicef`

**Countries:** `egypt` `saudiarabia` `uae` `usa` `france` `germany`

**High-Value Generic Names:** `news` `media` `sports` `music` `movies` `tv` `radio` `official` `verified` `premium` `vip`

---

## 3. Blocked Terms

Prohibited terms blocked regardless of availability, to protect platform
integrity, user safety, brand reputation, legal compliance, and community
standards.

**Adult & Sexual Content:** `sex` `porn` `porno` `xxx` `adult` `nude` `naked` `escort` `camgirl` `camboy` `onlyfans`

**Explicit Profanity:** `fuck` `fucking` `shit` `bullshit` `bitch` `asshole` `motherfucker` `bastard` `dick` `pussy` `cunt` `slut` `whore`

**Hate & Abuse:** `racist` `terrorist` `nazism` `nazi` `hitler` `genocide` `extremist`

**Violence & Criminal Activity:** `murder` `killer` `killerforhire` `assassin` `terror` `bomb` `explosive` `cartel` `druglord`

**Fraud & Scam:** `scam` `fraud` `fakeaccount` `phishing` `hacker` `hacking` `cracker` `carding` `stealer`

**Illegal Substances:** `cocaine` `heroin` `meth` `methamphetamine` `ecstasy` `drugdealer`

**Self-Harm:** `suicide` `selfharm` `killmyself` `cutting`

**Impersonation Terms:** `supportteam` `customersupport` `officialsupport` `verificationteam` `securityteam`

**Misleading Authority Terms:** `adminteam` `staffteam` `modteam` `officialaccount` `verifiedaccount`

Additional categories (emerging abuse terms, regional profanity,
country-specific prohibited terms, regulatory requirements, platform abuse
patterns) may be added over time.
