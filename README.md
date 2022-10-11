# <img src="https://raw.githubusercontent.com/feathericons/feather/master/icons/package.svg" width="45"/>&nbsp;<span style="font-size: 36px"> Packj flags malicious/risky open-source packages</span> 

Add *Packj* audit to your workflow to flag malicious and other "risky" open-source software dependencies. Packj (pronounced _package_) is an open-source [tool](https://github.com/ossillate-inc/packj) that powers our large-scale security analysis platform [Packj.dev](https://packj.dev) to continuously vets packages and provides free reports.

[![GitHub Stars](https://img.shields.io/github/stars/ossillate-inc/packj?style=social)](https://github.com/ossillate-inc/packj/stargazers) [![Discord](https://img.shields.io/discord/910733124558802974?label=Discord)](https://discord.gg/8hx3yEtF) ![](https://img.shields.io/badge/status-beta-yellow)

# How it works #

Packj can audit **PyPI, NPM, and RubyGems** packages. It performs:
- static code analysis to check entire package code for use of filesystem, network, and process APIs (e.g., `connect`, `exec`),
- metadata analysis to check for attributes such as release timestamps, author email, downloads, dependencies, and 
- dynamic analysis to check the runtime behavior of the package (and all dependencies) by installing them analysing traces for system calls (e.g., `open()`, `fork()`). 

Packages with expired email domains, large release time gap, sensitive APIs, etc. are flagged as risky for [security reasons](#risky-attributes).

# Supported threats #

The design of Packj is guided by our study of 651 malware samples of documented open-source software supply chain attacks. Specifically, we have empirically identified a number of risky code and metadata attributes that make a package vulnerable to supply chain attacks. 

|Threat                                              |  Reason                                                    |
|     :-:                                            |   :-:                                                |
|  Old or abandonded packages  | Old or unmaintained packages do not receive security fixes |
|  Use of sensitive APIs, such as `exec` and `eval`  | Malware uses APIs from the operating system or language runtime to perform sensitive operations (e.g., read SSH keys) |
|  Invalid or no email addresses of the contributors | Incorrect or missing email addresses suggest lack of 2FA |
|  Presence and validity of public source repo | Absence of a public repo means no easy way to audit or review the source code publicly |

Full list of the attributes we track can be viewed at [config.yaml](https://github.com/ossillate-inc/packj/blob/main/packj/config.yaml)

# Customizing alerts #

Packj can be customized to ONLY send alerts that apply to your threat model. Simply create a `.packj.yaml` file in your repo, and enable/disable alerts that matter to you.

# How it works

- It first downloads the metadata from the registry using their APIs and analyze it for "risky" attributes.
- To perform API analysis, the package is downloaded from the registry using their APIs into a temp dir. Then, packj performs static code analysis to detect API usage. API analysis is based on [MalOSS](https://github.com/osssanitizer/maloss), a research project from our group at Georgia Tech.
- Vulnerabilities (CVEs) are checked by pulling info from OSV database at [OSV](https://osv.dev)
- Python PyPI and NPM package downloads are fetched from [pypistats](https://pypistats.org) and [npmjs](https://api.npmjs.org/downloads)
- Dynamic analysis is performed by installing the package under `strace` tool, which uses `ptrace` system calls underneath.
- All risks detected across dependencies in your repo are aggregated, formatted, and reported as a comment on the pull request.
