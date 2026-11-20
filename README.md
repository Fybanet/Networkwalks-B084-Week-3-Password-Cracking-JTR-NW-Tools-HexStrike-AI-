# Networkwalks B084 | Week 3 | Password Cracking (JTR, NW Tools & HexStrike AI)

Cybersecurity internship at Networkwalks. This repo covers Week 3: cracking the password of a protected PDF using three different approaches, plus setting up an AI-assisted security toolkit (HexStrike) along the way.

> **Disclaimer:** All cracking here was done on a PDF file provided for this lab (`My Locked PDF1.pdf` / `networkwalks_flag1.pdf`). This is educational practice on files I was given permission to test, not an attack on anything I don't own or wasn't authorised to work on.

## Modules

| Module | Topic | Tool(s) |
|---|---|---|
| W3-PM1 | Password cracking with JTR | John the Ripper, Johnny (GUI) |
| W3-PM2 | Password cracking with Networkwalks tools | NW Hash Calculator, NW Password Cracker |
| W3-Optional | AI-assisted password cracking | HexStrike AI MCP + Claude Desktop + JTR |

## W3-PM1: Password cracking with JTR

I used `pdf2john` (via the Online Hash Crack site) to pull the hash out of the locked PDF, then fed it into Johnny, the GUI for John the Ripper, on Windows.

Steps I followed:
1. Uploaded the locked PDF to a PDF hash extractor to get a `$pdf$...` hash.
2. Saved the hash into a text file (`hash1.txt`).
3. Opened Johnny, pointed it at my `john.exe`, and loaded `hash1.txt`.
4. Ran the attack with Johnny's default wordlist.

The password was cracked almost immediately: `password1`. Entering it into the PDF unlocked it and revealed the flag.

## W3-PM2: Password cracking with Networkwalks tools

Same PDF, but this time using Networkwalks' own browser-based tools instead of installing anything:

1. [NW Hash Calculator](https://networkwalks.com/hash-calculator/) — uploaded the PDF and it extracted the same `$pdf$...` hash, all done locally in the browser.
2. [NW Password Cracker](https://networkwalks.com/password-cracker/) — pasted the hash in and ran a dictionary attack against its built-in wordlist.

It found the same password, `password1`, near the top of the list, confirming the result from W3-PM1 with a completely different tool.

**Takeaway from both modules:** the same weak password cracked almost instantly with two unrelated tools and two different wordlists. That's the whole point of the lab — a common password gives an attacker almost no resistance, no matter which cracking tool they reach for.

## W3-Optional: Cracking the same PDF with AI (HexStrike MCP + Claude)

For the optional module I set up HexStrike AI as an MCP server so Claude Desktop could run John the Ripper for me through plain-English prompts, instead of me driving the GUI myself.

### Setting it up

1. Installed Claude Desktop on my Kali VM via the `aaddrick/claude-desktop-debian` APT repository.
2. Cloned HexStrike AI from GitHub, made a Python virtual environment, and installed its requirements.
3. Started the server with `python hexstrike_server.py`. It came up fine and listened on port 8888 (see `Hexstrike_server.png`).
4. Edited Claude Desktop's `claude_desktop_config.json` to add HexStrike as an MCP server, pointing at the virtual environment's Python and the `hexstrike_mcp.py` script.
5. Restarted Claude Desktop and confirmed the server showed as running.

### The Python version problem I ran into

This is the part that actually took the most time, so I'm documenting it properly rather than glossing over it.

When I first ran `pip3 install -r requirements.txt`, it hung for a long time downloading and re-downloading `angr` over and over. Each attempt tried a slightly older version number (9.2.212, then 9.2.211, then 9.2.209, and so on), building each one from source before giving up and trying the next. It never actually failed with a clear error, it just kept going in circles.

The cause turned out to be my Python version. My system was running **Python 3.14**, which is brand new, and `angr` (one of HexStrike's binary-analysis dependencies) hadn't been built for it yet. `pip` doesn't say "your Python is too new" in plain words, it just quietly tries older and older package versions hoping one will fit, which is why it looked like it was stuck.

To fix it properly rather than guessing, I checked when `angr` was actually released and which Python versions were current at that time. That pointed me to **Python 3.12** as a version that angr's release window actually supported. So instead of using my system's default Python, I:

```bash
pip install uv
uv venv --python 3.12 hexstrike_env
source hexstrike_env/bin/activate
uv pip install -r requirements.txt
```

`uv` downloaded Python 3.12 on its own and built the environment around it. With the right Python version in place, every package installed cleanly on the first try, including `angr`, and `hexstrike_server.py` ran without any further issues.

**Lesson:** when a dependency's release date is older than your interpreter, pip's backtracking can look like it's frozen when it's really just failing silently, version after version. Matching the Python version to when the package was actually released fixed it in one go.

### Using it

Once the MCP server was running and connected, I asked Claude directly:

- *"Check if John the Ripper is installed in this HexStrike MCP and show me its version"* — confirmed John the Ripper (Jumbo build) was available.
- *"Please calculate the hash value of this PDF file: /home/kali/Desktop/networkwalks_flag1.pdf"* — Claude ran the hash extraction for me and returned the MD5/SHA1/SHA256 values and the PDF hash.
- *"Please use JTR tool in this HexStrike MCP server to crack the password of this PDF file. Use the rockyou.txt wordlist dictionary."* — Claude ran John the Ripper against `rockyou.txt` and returned the cracked password, `password1`, in under a second, since it's near the top of that wordlist.

Opening the PDF with that password revealed the flag, matching the result from the two manual methods above.

## Key takeaways

- Three completely different approaches (manual JTR/Johnny, a browser-based tool, and an AI-driven MCP workflow) all cracked the same weak password in seconds. Weak passwords don't need a sophisticated attacker, they just need any cracking tool and a common wordlist.
- Letting an AI assistant run security tools through MCP is powerful, but it's only as good as the environment underneath it. A version mismatch I didn't notice at first (Python 3.14 vs. a dependency built for 3.12) nearly stopped the whole setup before it started.
- When pip looks stuck "backtracking" on a dependency, it's usually a sign of an environment mismatch worth investigating rather than something to wait out.

## Evidence

| Evidence | Description |
|---|---|
| `Hexstrike_server.png` | HexStrike AI server starting up and listening on port 8888 |
| `locked1.png` | First locked PDF cracked and flag revealed |
| `locked2.png` | Second locked PDF cracked and flag revealed |
| `locked3.png` | Third locked PDF cracked and flag revealed |


```

## Author

**[Iwunze Ogochukwu]** | Cybersecurity, Networkwalks B084 | GitHub: [Fybanet](https://github.com/Fybanet)
