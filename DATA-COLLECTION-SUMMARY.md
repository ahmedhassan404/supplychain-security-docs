# ChainGuard Data Collection 

## 1. Data Collection Overview

Collect all inputs needed to analyze npm packages and detect supply‑chain threats.

### Main Data Goals

 Package metadata
 Dependency graphs
 Vulnerability records
 Static code features
 Labeled malicious vs. benign datasets

## 2. Key Data Sources

 Category                      Source  Endpoint                                          Purpose                                                 Auth                                 
 ----------------------------  ---------------------------------------------------------  ------------------------------------------------------  ------------------------------------ 
 Package Metadata              npm Registry API (registry.npmjs.org)                      Names, descriptions, versions, maintainers, timestamps  None                                 
 Download Trends               npm Downloads API                                          Weeklymonthly download counts                          None                                 
 Version & Maintainer Details  Derived from npm registry payloads                         Version history, release dates, maintainer emails       None                                 
 Repository Intelligence       GitHub API                                                 Stars, forks, issues, commits, contributor activity     GITHUB_TOKEN                         
 Vulnerabilities               NVD (services.nvd.nist.gov)                                CVE metadata, CVSS                                      NVD_API_KEY                          
                               OSV (api.osv.devv1query)                                 Open-source vulnerability records                       None                                 
                               GitHub Advisories (api.github.comadvisories)              GHSA alerts                                             GITHUB_TOKEN                         
                               OSS Index (ossindex.sonatype.orgapiv3component-report)  Component risk reports                                  OSS_INDEX_USERNAME + OSS_INDEX_TOKEN 
                               npm Security Advisories (npm audit)                        Registry-specific advisories                            None                                 

## 3. Dependency Graph Collection

 Parse `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`.
 Build direct + transitive dependency trees with depth tracking.
 Derive graph metrics (degree, centrality, path lengths).

## 4. Vulnerability Databases

Capture for each source

 VulnerabilityCVE ID
 Severity + CVSS
 Affected version ranges
 Fixedpatched versions
 Reference URLs

## 5. Static Code Analysis

### Steps

1. Download npm tarballs.
2. Extract source files and install scripts.
3. Scan for risky APIs (e.g., `eval`, `exec`), obfuscation markers, remote downloads.
4. Record code metrics and flags.

## 6. Labeled Dataset Collection

### Malicious Sources

 GitHub Security Advisories
 npm malwareremoval reports
 OSV entries
 OSS Index malicious findings
 Research datasets (e.g., Backstabber’s Knife Collection)

### Benign Sources

 Top downloaded npm packages
 Verified org scopes (e.g., `@google`, `@microsoft`)
 Long-lived, frequently updated packages

## 7. Data Storage (High Level)

Normalize and persist

 Packages, versions, maintainers
 Dependency edgesgraphs
 Vulnerability associations
 Static analysis features
 Labels (malicious  benign  suspicious)