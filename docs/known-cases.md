# Known Cases: Dependency Risk Walkthrough

This demo tells the Log4Shell story end to end — from the official advisory
record down to a pull request — as a stand-in for how you'd triage a critical
vulnerability in one of *your* dependencies.

## Case 1: Log4Shell — remote code injection in Log4j

All facts below are taken from the official GitHub Security Advisory record.

| Field | Official record |
| --- | --- |
| Advisory (GHSA) | **GHSA-jfh8-c2jp-5v3q** |
| CVE | CVE-2021-44228 |
| Title | Remote code injection in Log4j |
| Severity | **Critical** (CVSS 3.1 score: **10.0**) |
| Directly affected package | **`org.apache.logging.log4j:log4j-core`** (Maven) |
| Affected versions | Any Log4j version **prior to v2.15.0** (v1 branch is EOL and vulnerable to other RCE vectors) |
| First patched version | **2.15.0** (the original fix; see the follow-up below) |
| Recommended version | **2.16.0**, which disables JNDI by default and removes message lookup support |

### What the advisory actually says

> Apache Log4j2 <=2.14.1 JNDI features used in configuration, log messages,
> and parameters do not protect against attacker controlled LDAP and other
> JNDI related endpoints. An attacker who can control log messages or log
> message parameters can execute arbitrary code loaded from LDAP servers
> when message lookup substitution is enabled.

In plain terms: logging untrusted or user-controlled data — including things
like exception traces and authentication failures — with a vulnerable
version can give an attacker remote code execution in your application.

### Package-level detail worth remembering

Only `org.apache.logging.log4j:log4j-core` is *directly* affected. The
`log4j-api` artifact is not, but the advisory tells you to keep it at the
same version as `log4j-core` for compatibility. Backports of the fix were
also released as 2.3.1, 2.12.2 and 2.12.3.

## Case 2: The follow-up flaw, disclosed days later

Log4j **2.15.0 contained an earlier fix, but that patch did not disable
attacker-controlled JNDI lookups in all situations.** Days later this gap
was tracked as a second critical flaw:

| Field | Official record |
| --- | --- |
| Advisory (GHSA) | **GHSA-7rjr-3q55-vv33** |
| CVE | CVE-2021-45046 |
| Title | Incomplete fix for Apache Log4j vulnerability |
| Severity | **Critical** (CVSS 3.1 score: **9.0**) |
| Directly affected package | **`org.apache.logging.log4j:log4j-core`** (Maven) |
| Vulnerable range | >= 2.13.0, < 2.16.0 |
| First patched version | **2.16.0** |

The 2.15.0 fix was incomplete in certain non-default configurations: an
attacker controlling Thread Context Map (MDC) input data, where the logging
configuration uses a non-default Pattern Layout with a Context Lookup
(e.g. `$${ctx:loginId}`) or a Thread Context Map pattern (`%X`, `%mdc`,
`%MDC`), could still craft a JNDI Lookup pattern and achieve remote code
execution. Log4j 2.16.0 fixes this by removing message lookup support and
disabling JNDI functionality by default.

## The takeaway

1. **Find the official record.** The advisory ID, severity, affected package
   and first patched version all come from the record — not from memory or
   a tweet.
2. **Check whether the first patch actually finished the job.** In this case
   2.15.0 was the first patched release, but 2.16.0 is the one to actually
   target because of the follow-up flaw.
3. **Map it to your dependency graph.** The advisory is about a Maven
   package; the question for your repo is whether you consume it directly or
   transitively, and at what version.
4. **Then let the platform help you.** Dependabot alerts and security PRs
   are the operational half of this story — see the repo's Security tab
   during the demo.

*Sources: GitHub Security Advisory Database —
[GHSA-jfh8-c2jp-5v3q](https://github.com/advisories/GHSA-jfh8-c2jp-5v3q)
(CVE-2021-44228) and
[GHSA-7rjr-3q55-vv33](https://github.com/advisories/GHSA-7rjr-3q55-vv33)
(CVE-2021-45046).*
