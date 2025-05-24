---
published: true
title: Recon Tools For Bug Bounty
date: 2025-05-24 07:40:00 +0200
categories: Bug-Bounty
tags:
  - Recon
---
# Reconnaissance Tools for Penetration Testing and Bug Bounty

Reconnaissance is the foundational step in penetration testing and bug bounty hunting, aimed at gathering information about a target to identify potential vulnerabilities. It is divided into two primary types: **Passive Reconnaissance**, which leverages open-source intelligence (OSINT) without direct interaction with the target, and **Active Reconnaissance**, which involves direct interaction with the target system. This chapter provides a comprehensive overview of tools used for both types, with practical examples focusing on extracting files (e.g., JS, HTML, PHP, XML, PNG) and analyzing the target’s infrastructure.

## 1. Passive Reconnaissance Tools

Passive reconnaissance involves collecting information without interacting with the target, making it legally safe and ideal for the initial stages of penetration testing or bug bounty hunting. The following tools are commonly used:

### 1.1. Google Dorks (via tools like lopseg.com.br)

- **Description**: A technique using advanced Google search queries to uncover sensitive information such as API keys, configuration files, or exposed data.
- **Usage**: Ideal for discovering JS, PHP, XML, and PNG files, as well as sensitive endpoints.
- **Practical Example**:
    
    ```bash
    site:example.com filetype:js
    site:example.com inurl:(config | api) filetype:xml
    site:example.com filetype:png
    ```
    
    Tools like [lopseg.com.br](https://lopseg.com.br/google-dork) can simplify the creation of Dorks.
- **Notes**: Avoid excessive queries to prevent Google’s rate-limiting. Combine Dorks with tools like `waybackurls` to expand the scope of file discovery.

### 1.2. Shodan (with Favicon Hash)

- **Description**: A search engine for internet-connected devices, used to identify servers, open ports, and SSL certificates.
- **Usage**: Helps locate servers associated with a domain and discover exposed files via open ports. Favicon Hash can identify the origin IP.
- **Practical Example**:
    
    ```bash
    shodan search ssl:"example.com" 200
    ```
    
    To extract Favicon Hash:
    
    ```bash
    curl -s https://example.com/favicon.ico > favicon.ico
    ```
    
    Use [favicon-hash.kmsec.uk](https://favicon-hash.kmsec.uk/) to convert the favicon to a hash, then search in Shodan:
    
    ```bash
    shodan search http.favicon.hash:<hash_value>
    ```
    
    To extract files:
    
    ```bash
    waybackurls example.com | grep -E '\.js$|\.php$|\.xml$|\.png$' | httpx -mc 200 > files.txt
    ```
    
- **Notes**: Be cautious of honeypots identified by Shodan. Use [favicons.teamtailor-cdn.com](https://favicons.teamtailor-cdn.com/) to retrieve the target’s favicon.

### 1.3. crt.sh

- **Description**: A service for extracting subdomains from SSL certificate logs.
- **Usage**: Used for subdomain enumeration, which can then be scanned for associated files.
- **Practical Example**:
    
    ```bash
    curl -s "https://crt.sh/?q=domain%.$1&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u | tee -a subs_domain.txt
    cat subs_domain.txt | httpx -silent | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Highly effective for subdomain discovery. Combine outputs with `httpx` to verify live hosts.

### 1.4. urlscan.io

- **Description**: A service for analyzing websites and discovering subdomains and paths.
- **Usage**: Used to identify subdomains and associated files.
- **Practical Example**:
    
    ```bash
    curl -s "https://urlscan.io/api/v1/search/?q=domain:example.com&size=10000" | jq -r '.results[]?.page?.domain' | grep -E "^[a-zA-Z0-9.-]+\.example\.com$" | sort -u | tee urlscan_subs.txt
    cat urlscan_subs.txt | httpx -silent | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Requires an API key for full access. Combine with tools like `gau` for broader coverage.

### 1.5. Web Archive (waybackurls)

- **Description**: A tool to extract archived URLs from the Wayback Machine for a given domain.
- **Usage**: Used to find historical JS, PHP, XML, and PNG files stored in the archive.
- **Practical Example**:
    
    ```bash
    curl -s "http://web.archive.org/cdx/search/cdx?url=*.example.com/*&output=json&collapse=urlkey" | jq -r '.[1:][] | .[2]' | grep -Eo '([a-zA-Z0-9._-]+\.)?example\.com' | sort -u | tee webarcive_subs.txt
    cat webarcive_subs.txt | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' | httpx -mc 200 > files.txt
    ```
    
- **Notes**: Effective for discovering old or hidden files. Filter results with `httpx` to verify live URLs.

### 1.6. Maltego

- **Description**: A visual analysis tool for collecting and linking OSINT data, such as domains and IP addresses.
- **Usage**: Used to map relationships between subdomains and associated files.
- **Practical Example**:
    
    ```
    In Maltego:
    1. Create a "Domain" entity for "example.com".
    2. Use Transforms like "To Subdomains" and "To Files" to extract subdomains and files (JS, XML, PNG).
    ```
    
- **Notes**: Requires a free or paid account. Use OSINT transforms to avoid active interaction.

### 1.7. theHarvester

- **Description**: A tool for gathering emails, subdomains, and hosts from sources like Google, Bing, and Shodan.
- **Usage**: Used for subdomain enumeration and discovering associated files.
- **Practical Example**:
    
    ```bash
    theHarvester -d example.com -b google,bing,shodan -f output.html
    cat output.html | grep -oP '[\w.-]+\.example\.com' | sort -u | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Use multiple sources (-b all) for comprehensive results. Avoid excessive queries to prevent rate-limiting.

### 1.8. Amass

- **Description**: An open-source OWASP tool for subdomain enumeration using OSINT.
- **Usage**: Used to collect subdomains and identify associated files.
- **Practical Example**:
    
    ```bash
    amass enum -passive -d example.com -o subdomains.txt
    cat subdomains.txt | httpx -silent | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Use the passive mode (-passive) to avoid direct interaction.

### 1.9. Recon-ng

- **Description**: A modular framework for passive reconnaissance with modules for OSINT collection.
- **Usage**: Used to gather subdomains and identify exposed files.
- **Practical Example**:
    
    ```
    In Recon-ng:
    > use recon/domains-hosts/hackertarget
    > set SOURCE example.com
    > run
    > show hosts > subdomains.txt
    cat subdomains.txt | httpx -silent | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Requires additional module installation and API keys for full functionality.

### 1.10. SpiderFoot

- **Description**: An automated OSINT tool for collecting data from multiple sources.
- **Usage**: Used to map subdomains and associated files.
- **Practical Example**:
    
    ```bash
    spiderfoot -s example.com -m sfp_dnsresolve,sfp_filemeta -o output.json
    cat output.json | jq -r '.[] | select(.type=="FILE") | .data' | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: SpiderFoot’s web interface simplifies visualization.

### 1.11. Gau (Get All URLs)

- **Description**: A tool to fetch URLs from multiple sources, including Common Crawl, AlienVault, and Wayback Machine.
- **Usage**: Complements waybackurls for discovering archived JS, PHP, XML, and PNG files.
- **Practical Example**:
    
    ```bash
    gau example.com | grep -E '\.js$|\.php$|\.xml$|\.png$' | httpx -mc 200 > files.txt
    ```
    
- **Notes**: Faster than waybackurls for multi-source URL collection. Available on [GitHub](https://github.com/lc/gau).

## 2. Active Reconnaissance Tools

Active reconnaissance involves direct interaction with the target, making it more effective but requiring explicit permission to avoid legal issues. The following tools are commonly used:

### 2.1. Subdominator

- **Description**: A tool for subdomain enumeration using OSINT and active techniques.
- **Usage**: Used to enumerate subdomains and scan them for associated files.
- **Practical Example**:
    
    ```bash
    subdominator -d example.com -O subdomains.txt
    cat subdomains.txt | httpx -silent | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Available on [GitHub](https://github.com/RevoltSecurities/Subdominator). Use custom wordlists for better results.

### 2.2. BBot

- **Description**: An automated tool for subdomain enumeration and asset discovery.
- **Usage**: Used to identify subdomains and extract associated files.
- **Practical Example**:
    
    ```bash
    bbot -t example.com -p subdomain-enum -o subdomains.txt
    cat subdomains.txt | httpx -silent | waybackurls | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Highly effective for comprehensive reconnaissance. Verify scope before using active mode.

### 2.3. GoBuster

- **Description**: A tool for subdomain enumeration and directory scanning.
- **Usage**: Used to discover hidden files and directories.
- **Practical Example**:
    
    ```bash
    gobuster dns -d example.com -w /root/wordlists/seclists/Discovery/DNS/bug-bounty-program-subdomains-trickest-inventory.txt -o subs3.txt
    cat subs3.txt | httpx -silent | gobuster dir -u FUZZ -w /path/to/file_wordlist.txt -e | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Available on [GitHub](https://github.com/OJ/gobuster). Use SecLists wordlists for optimal results.

### 2.4. httpx

- **Description**: A tool for filtering live domains and URLs with additional metadata like titles and status codes.
- **Usage**: Used to verify subdomains and filter files.
- **Practical Example**:
    
    ```bash
    cat subdomains.txt | httpx -td -title -sc -ip > httpx_example.com.txt
    cat httpx_example.com.txt | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Use the `-rl` option to avoid rate-limiting issues.

### 2.5. Nmap

- **Description**: A network scanning tool for identifying open ports and services.
- **Usage**: Used to scan servers and discover files via open ports.
- **Practical Example**:
    
    ```bash
    nmap -sV -p 80,443 --script http-enum example.com -oN nmap_output.txt
    cat nmap_output.txt | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Use the `http-enum` script for directory enumeration. Obtain permission before scanning.

### 2.6. Dirsearch

- **Description**: A tool for directory and file enumeration on web servers.
- **Usage**: Used to find JS, PHP, XML, and PNG files.
- **Practical Example**:
    
    ```bash
    dirsearch -u https://example.com -e js,php,xml,png -o files.txt
    cat files.txt | httpx -mc 200 > live_files.txt
    ```
    
- **Notes**: Use custom wordlists for better results.

### 2.7. FFUF

- **Description**: A fast tool for directory and parameter fuzzing.
- **Usage**: Used to discover hidden files.
- **Practical Example**:
    
    ```bash
    ffuf -w /path/to/wordlist.txt -u https://example.com/FUZZ -e .js,.php,.xml,.png -o ffuf_output.json
    cat ffuf_output.json | jq -r '.results[] | .url' > files.txt
    ```
    
- **Notes**: Faster than GoBuster in some cases. Use `-mc 200` to filter successful responses.

### 2.8. Wfuzz

- **Description**: A tool for directory and parameter fuzzing using brute-force techniques.
- **Usage**: Used to discover hidden files and directories.
- **Practical Example**:
    
    ```bash
    wfuzz -w /path/to/wordlist.txt --hc 404 -u https://example.com/FUZZ -z list,js-php-xml-png
    cat wfuzz_output.txt | grep -E '\.js$|\.php$|\.xml$|\.png$' > files.txt
    ```
    
- **Notes**: Adjust `--hc` to filter unwanted responses.

## 3. Tips for Effective Reconnaissance

- **Tool Integration**: Use outputs from one tool (e.g., Amass or crt.sh) as inputs for another (e.g., httpx or dirsearch) for comprehensive results.
- **File Analysis**: After extracting JS files, use tools like `grep` or **JSFinder** to search for API keys or sensitive endpoints:
    
    ```bash
    grep -r "api_key\|token\|password" files/
    ```
    
- **Legal Compliance**: Obtain explicit permission before conducting active reconnaissance, especially in bug bounty programs.
- **Avoid Blocking**: Use delays (e.g., `-rl 10` in httpx) to avoid rate-limiting.
- **Wordlists**: Leverage wordlists like SecLists for directory and file enumeration.

## 4. Conclusion

Passive and active reconnaissance tools provide a robust framework for gathering information and identifying vulnerabilities. By combining these tools and analyzing their outputs carefully, testers can build a comprehensive picture of the target, increasing the chances of discovering security flaws. Subsequent Blog will discuss how to use this information to design ethical attacks and write effective bug bounty reports.
