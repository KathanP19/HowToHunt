# Jira Vulnerability Hunting

Jira is one of the most common internet-facing assets in corporate bug bounty scopes, and teams routinely leave instances exposed with open signup, public JQL search, default Seraph credentials, or stale versions that hand out the exact release number. Findings range from information disclosure (project names, issue content, user lists, roadmap/vendor data) to account compromise and RCE on outdated builds.

Related: [Subdomain Takeover](../Subdomain_Takeover/Subdomain_Takeover.md) for deprovisioned Jira CNAMEs · [Default Credentials](../Misconfigurations/Default_Credential_And_Admin_Panel.md)

## Hunting Checklist

- [ ] Locate in-scope instances: subdomains named `jira-`, `-jira`, `support-`, `helpdesk`, `issues`, `tickets`, plus Shodan/Censys fingerprints.
- [ ] Test unauthenticated reachability: hit `/secure/MyJiraHome.jspa`, `/browse/`, `/issues/?jql=` as an anonymous user — a 200 with real content means the whole instance may be public.
- [ ] Check for open self-signup at `/secure/Signup!default.jspa` (signup form rendered = anyone can get a foothold).
- [ ] Enumerate users via JQL probes: valid username → "No issues found", invalid username → explicit "user name(s) do not exist" error.
- [ ] Enumerate project keys with `/browse/<KEY>` and `/issues/?jql=project=<KEY>` — try well-known keys (`JRA`, `CON`, `AAP`, `SR`, `DEV`, `SEC`, `OPS`, `SUP`) plus company abbreviations.
- [ ] Try Seraph default/demo credentials (`admin:admin`, `jira:jira`, `demo:demo`, `testuser:1234`) against `/rest/api/2/myself`.
- [ ] Fingerprint the version at `/secure/AboutPage.jspa` (and `/status` on newer builds), then map that version to known CVEs — hunt for outdated instances, don't assume.
- [ ] Check for subdomain takeover: Jira subdomains still CNAME'd to an unsupported/deprovisioned `*.atlassian.net` site returning 404/"site unavailable".
- [ ] Pull the public issue feed and attachments for sensitive data: vendor names, internal hostnames, credentials pasted into issues, unfinished security reports.
- [ ] Run Jira-Lens for the full 25+ check battery, then manually validate anything reportable.

## Test Cases & Examples

### Unauthenticated signup / access probe

```http
GET /secure/Signup!default.jspa HTTP/1.1
Host: jira.target.com
```

If the response renders a "Create New Account" form, the instance allows anonymous registration. A "You do not have permission to access this page" body is the hardened response.

### User enumeration via JQL

```http
GET /issues/?jql=project%3DJRA%20AND%20reporter%3D%22jsmith%22 HTTP/1.1
Host: jira.target.com
```

- `jsmith` exists → page loads with "No issues found" (search executed).
- `jsmith` does not exist → error banner: `The following user name(s) do not exist: jsmith`.

Scriptable loop:

```bash
for u in $(cat users.txt); do
  curl -s "https://jira.target.com/issues/?jql=reporter%3D%22$u%22" \
    | grep -q "do not exist" || echo "VALID: $u"
done
```

Also check `/rest/api/2/user/picker?query=a` — anonymous 200s with user objects leak directories.

### Project / issue enumeration

```http
GET /browse/JRA HTTP/1.1
Host: jira.target.com
```

200 with a project summary = key exists (and may be browsable). Sweep keys with:

```bash
for k in JRA CON AAP DEV SEC OPS SUP HR FIN; do
  code=$(curl -s -o /dev/null -w '%{http_code}' "https://jira.target.com/browse/$k")
  echo "$k -> $code"
done
```

`/rest/api/2/project` on a misconfigured instance returns every project key/name in JSON.

### Seraph default credentials

```bash
curl -s -u admin:admin "https://jira.target.com/rest/api/2/myself"
# 200 with displayName/email = default creds work; 401 = locked out
```

### Version disclosure → outdated-instance hunting

```http
GET /secure/AboutPage.jspa HTTP/1.1
Host: jira.target.com
```

The About page prints the exact Jira version, build number, and Java version. Take the version string and check it against the Atlassian advisory history for unpatched issues — target instances several releases behind first.

### Subdomain takeover via unsupported-atlassian-jira CNAME

```bash
dig +short support.target.com CNAME
# e.g. target-support.atlassian.net — but GET returns 404 "this site is no longer available"
```

A CNAME pointing at a deprovisioned/unsupported Atlassian Jira site is a takeover candidate: the DNS record outlived the instance. Verify the 404/served-by-Atlassian fingerprint, then follow Atlassian's claim/support process and report dangling CNAME to the program.

### Bulk scan with Jira-Lens

```bash
python3 jira-lens.py -d jira.target.com
```

## Tools & Resources

- [Jira-Lens](https://github.com/MayankPandey01/Jira-Lens) [Jira-Lens 🔍 is a Python Based vulnerability Scanner for JIRA.This tool Performs 25+ Checks including CVE's and Multiple Disclosures on the Provided JIRA Instance]
- Jira vulnerabilities and how they are exploited in the wild - [thehackerish Blog](https://thehackerish.com/jira-vulnerabilities-and-how-they-are-exploited-in-the-wild/)
- Fingerprint at scale: Shodan (`http.favicon.hash`, "Jira"), Censys, or `xjira`/nuclei Jira templates against your subdomain list
- Author: **Mayank Pandey** — **[@mayank_pandey01](https://twitter.com/mayank_pandey01)**

## Report Tips

- Prove impact, not just exposure: show a valid-but-private user name from the JQL error, a real project summary from `/browse/<KEY>`, or an actual issue containing internal hostnames/credentials — screenshots of the public feed beat a bare "AboutPage is reachable".
- For default credentials, authenticate as the default admin and demonstrate read access to users/projects (or your own PoC user) — do not modify or delete anything; state that in the report.
- For outdated versions: attach the About page version disclosure and name the unpatched advisory class (privilege escalation/RCE) you verified relevant behavior for; don't cite a CVE you haven't demonstrated.
- For takeover: include the CNAME chain, the 404/invalid-site response from Atlassian's endpoint, and evidence you did not claim it yourself.
- Bundle the recon into one finding per instance ("unauthenticated Jira exposing user directory and N projects") with severity driven by what the issues actually contain.
