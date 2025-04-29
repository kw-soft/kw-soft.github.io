---
layout: post
title: Python Process Scanner
date: 2025-04-29
categories: security
tags: [Cybersecurity, Process Scanning, VirusTotal, Python]
image:
  path: /assets/img/pypscan-preview.png
  alt: PyPScan Tool Preview
---

*Python Process Scanner*

Hey folks! I’m stoked to introduce **PyPScan**, a neat project I’ve got up on [GitHub](https://github.com/kw-soft/PyPScan). Let’s dive into what this tool is and how it helps keep tabs on your Windows processes with a little help from VirusTotal.

*What’s PyPScan?*

PyPScan is a Python-based process scanner that checks out running processes on your Windows machine. It uses the VirusTotal API to analyze the executables, giving you a heads-up on anything that might be sketchy. It’s smart about saving time with local caching and keeps you in the loop with a live progress display.

*What It Does*

PyPScan does some pretty cool stuff to make process scanning smooth and informative:
- **Grabs Processes**: Uses Windows Management Instrumentation (WMI) to list all running processes with valid executable paths.
- **Hashes Files**: Creates a unique SHA-256 hash for each executable to identify it.
- **Checks VirusTotal**: Looks up each hash in VirusTotal’s database (if it’s not already cached) to see how many antivirus engines flag it as malicious versus the total reports.
- **Saves Progress**: Stores results in a `vt_cache.json` file right after each API call, so you don’t lose progress if the script stops.
- **Shows Live Updates**: Displays a dynamic console summary with how many API calls are done, an estimated time left, and counts of processes flagged as threats versus clean ones.
- **Flags Threats**: Lets you set a threshold for what counts as “potentially malicious” and lists those processes at the end.
- **Handles Errors Nicely**: If your API key’s invalid (HTTP 401), it exits with a clear message.

It’s built to be lightweight and respects VirusTotal’s rate limits with a 15-second delay between calls.

*Getting Started*

Ready to try it? Head to the [GitHub repo](https://github.com/kw-soft/PyPScan) for the full details. Here’s the quick rundown:

1. **Clone the Repo**:
   ```bash
   git clone https://github.com/kw-soft/PyPScan.git
   cd PyPScan
   ```

2. **Install Dependencies**:
   You’ll need Python 3.6+ and a couple of packages:
   ```bash
   pip install wmi requests
   ```

3. **Set Up the Config**:
   Edit the `config.json` file in the repo to add your VirusTotal API key (get a free one from [VirusTotal](https://www.virustotal.com)):
   ```json
   {
       "API_KEY": "YOUR_VIRUSTOTAL_API_KEY"
   }
   ```
   Pro tip: Add `config.json` to your `.gitignore` to keep your key private!

4. **Run the Scanner**:
   ```bash
   python main.py
   ```

The script will scan your processes, check VirusTotal, and show a live progress bar. When it’s done, you’ll get a report of any potentially malicious processes.

*Wrapping It Up*

PyPScan is my take on making process scanning easy and transparent for Windows users. It’s perfect for anyone curious about what’s running on their system or looking to spot potential threats. Give it a whirl on [GitHub](https://github.com/kw-soft/PyPScan), maybe toss it a star if you find it handy, and let me know how it goes!

Stay safe,  
Kevin