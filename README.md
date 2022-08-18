# <img src="https://raw.githubusercontent.com/feathericons/feather/master/icons/package.svg" width="45"/>&nbsp;<span style="font-size: 36px"> Packj flags malicious/risky open-source packages</span> 

*Packj* (pronounced package) is an auditing tool to vet your open-source software packages for "risky" attributes that make them vulnerable to supply chain attacks. This is the tool behind our large-scale security analysis platform [Packj.dev](https://packj.dev) that continuously vets packages and provides free reports.

[![GitHub Stars](https://img.shields.io/github/stars/ossillate-inc/packj?style=social)](https://github.com/ossillate-inc/packj/stargazers) [![Discord](https://img.shields.io/discord/910733124558802974?label=Discord)](https://discord.gg/8hx3yEtF) ![](https://img.shields.io/badge/status-beta-yellow)

## Auditing a package ##

Packj can audit PyPI, NPM, and RubyGems packages. It performs:
- static code analysis to check entire package code for use of filesystem, network, and process APIs (e.g., `connect`, `exec`),
- metadata analysis to check for attributes such as release timestamps, author email, downloads, dependencies, and 
- optionally, dynamic analysis to check the runtime behavior of the package (and all dependencies) by installing them analysing traces for system calls (e.g., `open()`, `fork()`). 

Packages with expired email domains, large release time gap, sensitive APIs, etc. are flagged as risky for [security reasons](#risky-attributes).

# How it works

- It first downloads the metadata from the registry using their APIs and analyze it for "risky" attributes.
- To perform API analysis, the package is downloaded from the registry using their APIs into a temp dir. Then, packj performs static code analysis to detect API usage. API analysis is based on [MalOSS](https://github.com/osssanitizer/maloss), a research project from our group at Georgia Tech.
- Vulnerabilities (CVEs) are checked by pulling info from OSV database at [OSV](https://osv.dev)
- Python PyPI and NPM package downloads are fetched from [pypistats](https://pypistats.org) and [npmjs](https://api.npmjs.org/downloads)
- Dynamic analysis is performed by installing the package under `strace` tool, which uses `ptrace` system calls underneath.
- All risks detected are aggregated and reported 

# Risky attributes #

The design of Packj is guided by our study of 651 malware samples of documented open-source software supply chain attacks. Specifically, we have empirically identified a number of risky code and metadata attributes that make a package vulnerable to supply chain attacks. 

For instance, we flag inactive or unmaintained packages that no longer receive security fixes. Inspired by Android app runtime permissions, Packj uses a permission-based security model to offer control and code transparency to developers. Packages that invoke sensitive operating system functionality such as file accesses and remote network communication are flagged as risky as this functionality could leak sensitive data.

Some of the attributes we vet for, include

| Attribute        |  Type    | Description                                              |  Reason                                                    |
|       :---:      |   :-:    |     :-:                                                  |   :-:                                                      |
|  Release date    | Metadata | Version release date to flag old or abandonded packages  | Old or unmaintained packages do not receive security fixes |
|  OS or lang APIs | Code     | Use of sensitive APIs, such as `exec` and `eval`         | Malware uses APIs from the operating system or language runtime to perform sensitive operations (e.g., read SSH keys) |
|  Contributors' email | Metadata | Email addresses of the contributors | Incorrect or invalid of email addresses suggest lack of 2FA |
|  Source repo | Metadata | Presence and validity of public source repo | Absence of a public repo means no easy way to audit or review the source code publicly |

Full list of the attributes we track can be viewed at [packj.yaml](https://github.com/ossillate-inc/packj/blob/main/packj.yaml)

These attributes have been identified as risky by several other researchers [[1](https://arxiv.org/pdf/2112.10165.pdf), [2](https://www.usenix.org/system/files/sec19-zimmermann.pdf), [3](https://www.ndss-symposium.org/wp-content/uploads/ndss2021_1B-1_23055_paper.pdf)] as well. 
