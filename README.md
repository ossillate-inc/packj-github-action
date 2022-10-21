# <img src="https://raw.githubusercontent.com/feathericons/feather/master/icons/package.svg" width="45"/>&nbsp;<span style="font-size: 36px"> Packj flags malicious/risky open-source dependencies</span> 

Add *Packj* to your workflow to audit your pull requests for malicious/risky NPM/PyPI/Ruby dependencies. 

Powered by our open-source tool [Packj](https://github.com/ossillate-inc/packj).

[![GitHub Stars](https://img.shields.io/github/stars/ossillate-inc/packj?style=social)](https://github.com/ossillate-inc/packj/stargazers) [![Discord](https://img.shields.io/discord/910733124558802974?label=Discord)](https://discord.gg/8hx3yEtF) ![](https://img.shields.io/badge/status-beta-yellow)

# Example usage #

```yaml
# This is a basic workflow to help you get started with Actions
name: Packj security audit

# Controls when the workflow will run
on:
  pull_request:
    branches:
      - main

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:

  # This workflow contains a single job called "packj-audit"
  packj-security-audit:

    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:

      # Audit 
      - name: Audit dependencies
        uses: ossillate-inc/packj-github-action@0.0.4-beta
        with:
          # TODO: replace with your dependency files in the repo
          DEPENDENCY_FILES: pypi:requirements.txt,npm:package.json,rubygems:Gemfile
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs ###
- **DEPENDENCY_FILES** -- _Packj takes open-source dependency files as input (e.g., npm:package.json). Multiple comma-separated files could be specified. Please see example usage above.
- **REPO_TOKEN**:  -- specific github token, `secrets.GITHUB_TOKEN` is used by default (Github Action Runner)

### Output ###

Packj will comment on the PR if any risky dependencies are found. See example [PR run](https://github.com/ossillate-inc/packj-github-action-demo/pull/3).

# How it works #

Packj can audit **PyPI, NPM, and RubyGems** packages. It performs:
- static code analysis to check entire package code for use of filesystem, network, and process APIs (e.g., `connect`, `exec`),
- metadata analysis to check for attributes such as release timestamps, author email, downloads, dependencies, and 
- dynamic analysis to check the runtime behavior of the package (and all dependencies) by installing them analyzing traces for system calls (e.g., `open()`, `fork()`). 

Packages with expired email domains, large release time gap, sensitive APIs, etc. are flagged as risky for [security reasons](https://github.com/ossillate-inc/packj/blob/main/packj/audit/README.md#risky-attributes).

# Supported threats #

The design of Packj is guided by our study of 651 malware samples of documented open-source software supply chain attacks. It scans for 40+ risky code and metadata attributes that make a package vulnerable to supply chain attacks. 

<details>
  <summary>
    <strong>Show examples of risky packages that are flagged by Packj.</strong>
  </summary>
  <table>
    <tr>
      <th>Risk</th>
      <th>Reason</th>
    </tr>
    <tr>
      <td>Packages impersonating popular packages</td>
      <td>Bad actors carry out typo-squatting attacks by publishng malicious packages that impersonate popular packages (e.g., lodash vs loadash)</td>
    </tr>
    <tr>
      <td>Old or abandonded packages</td>
      <td>Old or unmaintained packages do not receive security fixes</td>
    </tr>
    <tr>
      <td>Package using of sensitive APIs, such as <b>exec</b> and <b>eval</b></td>
      <td>Malware uses APIs from the operating system or language runtime to perform sensitive operations (e.g., read SSH keys)</td>
    </tr>
    <tr>
      <td>Packages with invalid or no email addresses of the contributors</td>
      <td>Incorrect or missing email addresses suggest lack of 2FA on the account, which makes it easier for bad actors to hijack package ownership</td>
    </tr>
    <tr>
      <td>Packages with invalid or no public source repo</td>
      <td>Absence of a public repo means no easy way to audit or review the source code publicly</td>
    </tr>
        <tr>
      <td>Packages containing known vulnerabilities</td>
      <td>Known security vulnerabilities (CVEs) in package code could be exploited by bad actors</td>
    </tr>
  </table>
  Full list of risks we track can be viewed at <a href="https://github.com/ossillate-inc/packj/blob/main/packj/config.yaml">github.com/ossillate-inc/packj/config.yaml</a>
</details>

# Customizing alerts #

Packj can be customized to ONLY send alerts that apply to your threat model. Simply add a `.packj.yaml` file in your repo based on our template (below) and enable/disable alerts that matter to you.

<details>
  <summary>
    <strong>Show .packj.yaml template.</strong>
  </summary>
  <pre><code class="lang-yaml">
  
	#
	# Audit policies
	#
	audit:
	
	  #
	  # Risk alerts (enable or disable according to your threat model)
	  # 
	  alerts:
	
	    #
	    # category: malicious packages (publicly known and unknown)
	    #
	    malicious:
	      contains known malware:
	        - reason: package is known to contain a dangerous malware
	        - enabled: true
	      typo-squatting or repo-jacking package:
	        - reason: package impersonates another popular package to propagate malware
	        - enabled: true
	
	    #
	    # alert category: packages vulnerable to code exploits
	    #
	    vulnerable:
	      contains known vulnerabilities:
	        - reason: known vulnerabilities (CVEs) in package code could be exploited
	        - enabled: true
	
	    #
	    # packages with undesirable or "risky" attributes
	    #
	    undesirable:
	      package is old or abandoned:
	        - reason: old or abandoned packages receive no security updates and are risky
	        - enabled: true
	
	      invalid or no author email:
	        - reason: a package with lack of or invalid author email suggests 2FA not enabled
	        - enabled: true
	
	      invalid or no homepage:
	        - reason: a package with no or invalid homepage may not be preferable
	        - enabled: false
	
	      no source repo:
	        - reason: lack of public source repo may suggest malicious intention
	        - enabled: true
	
	      fewer downloads:
	        - reason: a package with few downloads may not be preferable
	        - enabled: true
	
	      no or insufficient readme:
	        - reason: a package with lack of documentation may not be preferable
	        - enabled: false
	
	      fewer versions or releases:
	        - reason: few versions suggest unstable or inactive project
	        - enabled: true
	
	      too many dependencies:
	        - reason: too many dependencies increase attack surface
	        - enabled: false
	
	      version release after a long gap:
	        - reason: a release after a long time may indicate account hijacking
	        - enabled: false
	
	      contains custom installation hooks:
	        - reason: custom installation hooks may download or execute malicious code
	        - enabled: false # WIP
	
	      #
	      # type: repo stats
	      #
	      few source repo stars:
	        - reason: a package with few repo stars may not be preferable
	        - enabled: false
	
	      few source repo forks:
	        - reason: a package with few repo forks may not be preferable
	        - enabled: false
	
	      forked source repo:
	        - reason: a forked copy of a popular package may contain malicious code
	        - enabled: true
	
	      #
	      # type: APIs and permissions
	      #
	      generates new code:
	        - reason: package generates new code at runtime, which could be malicious
	        - enabled: false
	      forks or exits OS processes:
	        - reason: package spawns new operating system processes, which could be malicious
	        - enabled: false
	      accesses obfuscated (hidden) code:
	        - enabled: true
	      accesses environment variables:
	        - enabled: false
	      changes system/environment variables:
	        - enabled: false
	      accesses files and dirs:
	        - enabled: false
	      communicates with external network:
	        - enabled: false
	      reads user input:
	        - enabled: false
	
	sandbox:
	  rules:
	    #
	    # File system (allow or block accesses to file/dirs)
	    #
	    #   ~/ represents home dir
	    #   . represents cwd dir
	    #
	    # NOTE: only ONE 'allow' and 'block' lines are allowed
	    #
	    fs:
	      # TODO: customize as per your threat model
	
	      # block access to home dir and all other locations (except the ones below)
	      block: ~/, /
	      allow: ., ~/.cache, ~/.npm, ~/.local, ~/.ruby, /tmp, /proc, /etc, /var, /bin, /usr/include, /usr/local, /usr/bin, /usr/lib, /usr/share, /lib
	
	    #
	    # Network (allow or block domains/ports)
	    #
	    # NOTE: only ONE 'allow' and 'block' lines are allowed
	    #
	    network:
	
	      # TODO: customize as per your threat model
	
	      # block all external network communication (except the ones below)
	      block: 0.0.0.0
	
	      # For installing PyPI, Rubygems, and NPM packages
	      allow: pythonhosted.org:443, pypi.org:443, rubygems.org:443, npmjs.org:0, npmjs.com:0
  </code></pre>
</details>
