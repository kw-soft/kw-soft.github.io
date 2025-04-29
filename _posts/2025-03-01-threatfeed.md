---
layout: post
title: Discord ThreatFeed HQ
date: 2025-03-01
categories: security
tags: [Cybersecurity, Threat Intelligence, Discord, Python]
image:
  path: /assets/img/threatfeed.png
  alt: ThreatFeed HQ 
---

*ThreatFeed HQ*

Hey there! I’m thrilled to tell you about **ThreatFeed HQ**, a cool project I’ve got running on [GitHub](https://github.com/kw-soft/ThreatIntelligence). It’s all about keeping you in the loop with the latest security news, and the best part? You can get real-time updates by joining our [Discord server](https://discord.com/invite/YOUR_DISCORD_INVITE_LINK)! Let’s check out what this tool does and why you’ll want to hop on board.

*What’s ThreatFeed HQ?*

ThreatFeed HQ is an RSS feed aggregator that pulls security advisories and news from top sources like Sophos, Cisco, Microsoft, and more, then sends them straight to Discord channels via webhooks. It’s your one-stop shop for staying updated on threat intelligence, and our Discord server is where it all comes to life with real-time posts.

*What It Does*

This tool is built to make threat intelligence accessible and organized:
- **Grabs News Automatically**: Pulls RSS feeds from sources like Sophos, Cisco, CVEFeed, and HackerNews at set intervals.
- **Sorts and Posts**: Arranges news entries chronologically and sends them to a global Discord channel plus specific vendor channels (e.g., one for Sophos, one for Cisco).
- **Skips Duplicates**: Uses a JSON file to track what’s been posted, so you don’t see the same news twice.
- **Keeps It Tidy**: Posts updates in a clean, structured format via Discord webhooks.
- **Logs Everything**: Saves logs for debugging or checking what’s going on.

By joining our [Discord server](https://discord.com/invite/BgUCmYP3px), you’ll see these updates as they happen, from sources like ProjectZero, CheckPoint, and BleepingComputer, all in one place!

*Getting Started*

Want to set it up yourself or just join the action? Here’s how:
- **Join Our Discord**: Hop into our [ThreatFeed HQ Discord server](https://discord.com/invite/BgUCmYP3px) to get real-time security updates from all our feeds. It’s the easiest way to stay informed!
- **Run It Yourself**: If you want to host your own instance, check out the [GitHub repo](https://github.com/kw-soft/ThreatIntelligence) for full details. Quick steps:
  1. **Clone the Repo**:
     ```bash
     git clone https://github.com/KW-Soft/ThreatIntelligence.git
     cd ThreatIntelligence
     ```
  2. **Install Dependencies**:
     You’ll need Python 3.7+ and the required packages:
     ```bash
     pip install -r requirements.txt
     ```
  3. **Set Up Webhooks**:
     Create Discord webhooks in your server and add them to `config.py`. For example:
     ```python
     GLOBAL_DISCORD_WEBHOOK = "https://discord.com/api/webhooks/YOUR_GLOBAL_WEBHOOK_ID/YOUR_GLOBAL_WEBHOOK_TOKEN"
     FEED_DISCORD_WEBHOOKS = {
         "SophosFeed": ["https://discord.com/api/webhooks/YOUR_SOPHOS_WEBHOOK_ID/YOUR_SOPHOS_WEBHOOK_TOKEN"],
         "CiscoFeed": ["https://discord.com/api/webhooks/YOUR_CISCO_WEBHOOK_ID/YOUR_CISCO_WEBHOOK_TOKEN"]
     }
     ```
  4. **Start the Aggregator**:
     ```bash
     python main.py
     ```

The README has more on tweaking polling intervals or adding new feeds.

*Why Join Our Discord?*

Our [Discord server](https://discord.com/invite/BgUCmYP3px) is where ThreatFeed HQ shines. You’ll get:
- Instant updates from trusted sources like Microsoft, Schneier, and CVEFeed.
- A community of security enthusiasts to chat with.
- The chance to suggest new feeds or features directly to us!

Plus, it’s free to join, and you’ll stay ahead of the curve on security threats. Don’t miss out—[join us today](https://discord.com/invite/BgUCmYP3px)!

*Wrapping It Up*

ThreatFeed HQ is my way of bringing threat intelligence to your fingertips, and our Discord server is the heart of it all. Whether you’re a security pro or just curious, come hang out with us on [Discord](https://discord.com/invite/BgUCmYP3px) or explore the project on [GitHub](https://github.com/kw-soft/ThreatIntelligence). If you like it, give it a star and share your thoughts!

Stay safe,  
Kevin