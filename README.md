# Log Analysis Tool: Brute Force Detection
 
A Python tool that analyses SSH authentication logs and detects possible brute force attacks. It extracts failed logins, groups them by source IP address, flags IPs that make repeated failed attempts within a short time window, and produces a text report and a chart of the top attackers.
 
**Authors:** Daniel Vintila and Callum Matthews
**Built with:** Python 3, matplotlib
 
---
 
## What It Does
 
1. Reads an SSH authentication log line by line.
2. Extracts the timestamp, source IP address, and event type (failed login, accepted login, or other).
3. Collects the timestamps of all failed logins for each IP address.
4. Uses a sliding time window to detect brute force behaviour: any IP with **5 or more failed logins within 10 minutes** is flagged as an incident.
5. Calculates the total number of failed logins per IP address.
6. Writes a report to a text file and prints the results to the terminal.
7. Saves a bar chart of the five IP addresses with the most failed logins.
---
 
## How Detection Works
 
For each IP address, the failed login timestamps are sorted in chronological order. Starting from each timestamp, the tool extends a 10 minute window forward and counts how many failed logins fall inside it.
 
- If the count is **5 or more**, the window is recorded as an incident with the IP, the number of attempts, and the first and last attempt times.
- The tool then skips past that cluster so the same burst of attempts is not reported twice.
- If the count is below 5, it moves to the next timestamp and checks again.
An IP that attacks in several separate bursts will appear once per burst in the incident list.
 
| Setting | Value | Where to change it |
| --- | --- | --- |
| Time window | 10 minutes | `window = timedelta(minutes=10)` |
| Alert threshold | 5 failed attempts | `if count >= 5:` |
| Log file | `CA1_project.log` | `LOGFILE` variable |
 
---
 
## Requirements
 
- Python 3.x
- matplotlib
Install the dependency with:
 
```bash
pip install matplotlib
```
 
---
 
## Usage
 
1. Place your log file in the same folder as the script, or update the `LOGFILE` variable to point to it.
2. Run the script:
```bash
python Assignment1.py
```
 
---
 
## Input Format
 
The tool expects standard Linux SSH auth log lines, for example:
 
```
Mar 14 02:11:45 server sshd[1234]: Failed password for root from 203.0.113.7 port 52344 ssh2
```
 
- The first three fields are read as the timestamp (month, day, time).
- Syslog timestamps do not include a year, so the tool assumes **2025**. Change the year in `parse_auth_line` if your log is from a different year.
- Lines containing `Failed password` are treated as failed logins.
- The IP address is taken from the word that follows `from`.
---
 
## Output
 
**Terminal and `brute_force_attempts.txt`**
 
The report contains two sections. The first lists each detected incident:
 
```
IP: 203.0.113.7, Failed Attempts: 12, First Attempt: 2025-03-14T02:11:45, Last Attempt: 2025-03-14T02:16:02
```
 
The second lists the total failed logins for every IP address:
 
```
IP: 203.0.113.7, Failed Attempts: 48
```
 
**Chart**
 
A bar chart of the top 5 IP addresses by total failed logins is saved to the project folder as `Top_5_Attackrs.png`.
 
The IP addresses above are documentation examples, not real data.
 
---
 
## Project Files
 
| File | Description |
| --- | --- |
| `Assignment1.py` | Main script: parsing, detection, report, and chart |
| `CA1_project.log` | Sample SSH authentication log used for analysis |
| `brute_force_attempts.txt` | Generated report of detected incidents and totals |
| `Top_5_Attackrs.png` | Generated chart of the top 5 attacking IP addresses |
 
---
 
## Limitations and Possible Improvements
 
- Only SSH password failures are counted. Failed public key attempts and other log formats are not detected.
- The year is fixed at 2025 and the log file name is hard coded.
- The threshold and window are fixed in the code and could be made command line arguments.
- Successful logins are parsed but not yet used. Flagging an accepted login that follows a burst of failures from the same IP would highlight a possible successful compromise.
- Results could be exported as CSV or JSON for use in other tools.
---
 
## Purpose
 
Created as part of coursework for a Cybercrime and IT Security degree at South East Technological University (SETU), to practise log analysis and the detection of brute force attacks.
