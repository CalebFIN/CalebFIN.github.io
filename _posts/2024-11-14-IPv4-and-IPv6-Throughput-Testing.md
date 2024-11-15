---
title: IPv4 and IPv6 Throughput Testing
date: 2024-11-14 22:28:00 -0500
categories: [Benchmarking, Ipv6]
tags: [data, networking]
---


Recently, I conducted an experiment to compare download speeds when using IPv4 versus IPv6. For anyone curious about the practical testing methods between these two protocols, I'd like to walk you through what I tested and what I found.

**The Setup**

For this quick test, I used `curl` to download a 1GB test file from Tele2's speed test server. I forced the download to occur over both IPv4 and IPv6 to see if there was any difference in download speeds. Here's how I set it up:

```bash
curl -6 http://speedtest.tele2.net/1GB.zip -o tempfile6
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1024M  100 1024M    0     0  6210k      0  0:02:48  0:02:48 --:--:-- 8125k
```
- **Total Download Time:** Approximately 2 minutes and 48 seconds
- **Average Speed:** Around 6.2 MB/s, peaking at 8.1 MB/s

```bash
curl -4 http://speedtest.tele2.net/1GB.zip -o ipv4temp
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1024M  100 1024M    0     0  4198k      0  0:04:09  0:04:09 --:--:-- 12.4M
```
The result:
- **Total Download Time:** Approximately 4 minutes and 9 seconds
- **Average Speed:** Around 4.2 MB/s, peaking at 12.4 MB/s

### Why the Difference?

I'm assuming are a few potential reasons for this speed difference:

1. **Network Prioritization** - Some networks prioritize IPv6 traffic since it’s the newer protocol and helps reduce the load on IPv4 resources?
2. **Directness of Route** - IPv6 may have taken a more direct route, which led to a faster download speed?


## IPERF3

### Test Scenarios

I conducted various tests using both IPv4 and IPv6, including:

- **Basic Tests:** Single connection download and upload tests.
- **Parallel Connections:** Tests with multiple parallel streams to saturate the connection.
- **Reverse Mode:** Testing upload speeds from my machine to the server.
- **Multi-Port Tests:** Utilizing multiple ports to test simultaneous connections.


```bash
iperf3 -4 -c speedtest.dal13.us.leaseweb.net -p 5201 -P 1
iperf3 -6 -c speedtest.dal13.us.leaseweb.net -p 5201 -P 1
iperf3 -4 -c speedtest.dal13.us.leaseweb.net -p 5201 -P 4
iperf3 -6 -c speedtest.dal13.us.leaseweb.net -p 5201 -P 4
iperf3 -4 -c speedtest.dal13.us.leaseweb.net -p 5201 -R
iperf3 -6 -c speedtest.dal13.us.leaseweb.net -p 5201 -R
iperf3 -4 -c speedtest.dal13.us.leaseweb.net -p 5201-5210 -P 4 -R
iperf3 -6 -c speedtest.dal13.us.leaseweb.net -p 5201-5210 -P 4 -R
iperf3 -4 -c speedtest.dal13.us.leaseweb.net -p 5201 -i 0.5 -P 4 -R
iperf3 -6 -c speedtest.dal13.us.leaseweb.net -p 5201 -i 0.5 -P 4 -R
```

## Results

| Test Scenario           | Protocol | Direction | Transfer (MB) | Bitrate (Mbits/sec) | Interval       |
|-------------------------|----------|-----------|---------------|---------------------|----------------|
| Basic Test              | IPv4     | Download  | 17.8          | 14.9                | 0.00-10.01     |
| Basic Test              | IPv6     | Download  | 17.0          | 14.3                | 0.00-10.01     |
| Parallel Connections    | IPv4     | Download  | 18.2          | 15.3                | 0.00-10.02     |
| Parallel Connections    | IPv6     | Download  | 17.1          | 14.4                | 0.00-10.01     |
| Reverse Mode            | IPv4     | Upload    | 105.0         | 87.4                | 0.00-10.06     |
| Reverse Mode            | IPv6     | Upload    | 157.0         | 130.0               | 0.00-10.09     |
| Multi-Port              | IPv4     | Upload    | 138.0         | 115.0               | 0.00-10.09     |
| Multi-Port              | IPv6     | Upload    | 167.0         | 139.0               | 0.00-10.12     |
| Detailed Interval       | IPv4     | Upload    | 119.0         | 99.4                | 0.00-10.05     |
| Detailed Interval       | IPv6     | Upload    | 180.0         | 150.0               | 0.00-10.08     |


Testing with a single server didn't provide enough information, so I expanded my tests to multiple servers. I used ChatGPT to help me write a script to test multiple sites, which took about four hours of testing.

### Servers Tested

I included a variety of [public servers](https://iperf3serverlist.net/) from different North American locations, such as:

- Servers in Dallas, New York, Chicago, Los Angeles, Miami, Phoenix, San Francisco, and Seattle.

### Test Methodology

For each server, I conducted the following tests:

- **IPv4 and IPv6 Basic Tests:** Measuring bandwidth with a single stream.
- **Reverse Mode Tests:** Testing upload speeds.
- **Incremental Parallel Connections:** Testing with 2 to 8 parallel streams.
- **Multi-Port Tests:** Utilizing multiple ports with maximum parallel connections.

I developed an in-depth benchmarking [script.](https://github.com/CalebFIN/Iperf3-Testset/blob/main/Long_Benchmark.py). 
- Since running the script took about an hour each time, I decided to create a [quicker benchmarking method](https://github.com/CalebFIN/Iperf3-Testset/blob/main/Quick_Benchmark.py).

```python
import subprocess
import json
import re
from datetime import datetime

# Constants
OUTPUT_FILE = "Quick_Benchmark_results.txt"
REPORT_FILE = "Quick_Benchmark_detailed_report.md"
INTERVAL = 0.3  # Reduced interval
MAX_PARALLEL = 6  # Fewer parallel tests

# Server list in "IP/HOST OPTIONS" format
SERVERS = [
    "bhs.proof.ovh.ca -p 5201-5210",
    "speedtest.nyc1.us.leaseweb.net -p 5201-5210",
    "speedtest.chi11.us.leaseweb.net -p 5201-5210"
]  # Reduced to 3 servers for speed

results = []

def run_command(command):
    try:
        print(f"Executing command: {command}")
        result = subprocess.run(command, shell=True, check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        output = result.stdout.decode('utf-8') + result.stderr.decode('utf-8')
        return output
    except subprocess.CalledProcessError as e:
        error_output = e.stdout.decode('utf-8') + e.stderr.decode('utf-8')
        return f"[ERROR] {error_output}"

def extract_stats(result):
    stats = {}
    # Extract bandwidth
    bandwidth_match = re.findall(r'\[SUM\].*?([0-9\.]+\s[MKG]bits/sec)', result)
    if not bandwidth_match:
        bandwidth_match = re.findall(r'0\.00-.*?sec.*?([0-9\.]+\s[MKG]bits/sec)', result)
    if bandwidth_match:
        stats['bandwidth'] = bandwidth_match[-1]
    else:
        stats['bandwidth'] = "N/A"
    # Extract jitter and packet loss (for UDP tests if applicable)
    jitter_match = re.search(r'([0-9\.]+)\sms\s+jitter', result)
    packet_loss_match = re.search(r'([0-9\.]+)%\s+packet\s+loss', result)
    if jitter_match:
        stats['jitter'] = jitter_match.group(1) + " ms"
    if packet_loss_match:
        stats['packet_loss'] = packet_loss_match.group(1) + "%"
    return stats

def calculate_average_bandwidth(tests, ip_version):
    total_bandwidth = 0.0
    count = 0
    for test in tests:
        if ip_version in test["test"]:
            bandwidth_str = test["stats"].get("bandwidth", "N/A")
            if bandwidth_str != "N/A":
                bandwidth_value = float(bandwidth_str.split()[0])
                unit = bandwidth_str.split()[1]
                # Convert bandwidth to Mbits/sec
                if unit == "Kbits/sec":
                    bandwidth_value /= 1000
                elif unit == "Gbits/sec":
                    bandwidth_value *= 1000
                total_bandwidth += bandwidth_value
                count += 1
    return total_bandwidth / count if count > 0 else "N/A"

# Main execution
def main():
    # Loop through each server and perform tests
    for server in SERVERS:
        server_name = server.split()[0]
        server_results = {"server": server_name, "tests": []}
        print(f"\nRunning tests for {server_name}...")

        # Run basic IPv4 test
        print(f"Running basic IPv4 test for {server_name}")
        ipv4_result = run_command(f"iperf3 -4 -c {server} -P 1 -i {INTERVAL}")
        server_results["tests"].append({
            "test": "basic_ipv4",
            "result": ipv4_result,
            "stats": extract_stats(ipv4_result)
        })

        # Run basic IPv6 test
        print(f"Running basic IPv6 test for {server_name}")
        ipv6_result = run_command(f"iperf3 -6 -c {server} -P 1 -i {INTERVAL}")
        server_results["tests"].append({
            "test": "basic_ipv6",
            "result": ipv6_result,
            "stats": extract_stats(ipv6_result)
        })

        # Incremental Parallel Connections Test
        for parallel in [2, 4, MAX_PARALLEL]:  # Limited parallel tests
            print(f"Running IPv4 test with {parallel} parallel connections on {server_name}")
            parallel_ipv4_result = run_command(f"iperf3 -4 -c {server} -P {parallel} -i {INTERVAL}")
            server_results["tests"].append({
                "test": f"parallel_ipv4_{parallel}",
                "result": parallel_ipv4_result,
                "stats": extract_stats(parallel_ipv4_result)
            })

            print(f"Running IPv6 test with {parallel} parallel connections on {server_name}")
            parallel_ipv6_result = run_command(f"iperf3 -6 -c {server} -P {parallel} -i {INTERVAL}")
            server_results["tests"].append({
                "test": f"parallel_ipv6_{parallel}",
                "result": parallel_ipv6_result,
                "stats": extract_stats(parallel_ipv6_result)
            })

        print(f"Completed tests for {server_name}. Results stored in memory.")
        results.append(server_results)

    # Write results to the output file in JSON format
    with open(OUTPUT_FILE, "w") as f:
        f.write(json.dumps(results, indent=4))

    # Generate the detailed report
    generate_report(results)
    print(f"\nAll tests completed! Results are stored in {OUTPUT_FILE} and {REPORT_FILE}")

def generate_report(data):
    report_content = "# Quick Benchmark Detailed Report\n\n"
    report_content += f"**Test Date:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n"

    # Introduction
    report_content += "## Introduction\n\n"
    report_content += "This report summarizes the quick iPerf network performance tests conducted on selected servers. The tests measure bandwidth for IPv4 and IPv6 connections under various parallel connections.\n\n"

    # Methodology
    report_content += "## Methodology\n\n"
    report_content += "The tests were performed using the iPerf3 tool with the following configurations:\n\n"
    report_content += f"- **Interval:** {INTERVAL} seconds\n"
    report_content += f"- **Maximum Parallel Connections:** {MAX_PARALLEL}\n"
    report_content += "- **Servers Tested:**\n"
    for server in SERVERS:
        report_content += f"  - {server}\n"
    report_content += "\nEach server was tested under the following scenarios:\n\n"
    report_content += "- Basic IPv4 and IPv6 tests\n"
    report_content += "- Parallel connections with 2, 4, and 6 streams\n\n"

    # Summary Table of Averages
    report_content += "## Summary of Results\n\n"
    report_content += "| Server | IPv4 Average Bandwidth (Mbits/sec) | IPv6 Average Bandwidth (Mbits/sec) |\n"
    report_content += "|--------|---------------------------|---------------------------|\n"

    # Calculate averages and add them to the summary table
    for server_result in data:
        server_name = server_result["server"]
        ipv4_avg = calculate_average_bandwidth(server_result["tests"], "ipv4")
        ipv6_avg = calculate_average_bandwidth(server_result["tests"], "ipv6")
        ipv4_avg_str = f"{ipv4_avg:.2f}" if isinstance(ipv4_avg, float) else "N/A"
        ipv6_avg_str = f"{ipv6_avg:.2f}" if isinstance(ipv6_avg, float) else "N/A"
        report_content += f"| {server_name} | {ipv4_avg_str} | {ipv6_avg_str} |\n"

    # Detailed Results for Each Server
    report_content += "\n## Detailed Test Results\n\n"
    for server_result in data:
        server_name = server_result["server"]
        report_content += f"### Results for {server_name}\n\n"
        report_content += "| Test | Bandwidth | Jitter | Packet Loss |\n"
        report_content += "|------|-----------|--------|-------------|\n"

        for test in server_result["tests"]:
            test_name = test["test"]
            stats = test["stats"]
            bandwidth = stats.get("bandwidth", "N/A")
            jitter = stats.get("jitter", "N/A")
            packet_loss = stats.get("packet_loss", "N/A")
            report_content += f"| {test_name} | {bandwidth} | {jitter} | {packet_loss} |\n"

        report_content += "\n"

    # Conclusion
    report_content += "## Conclusion\n\n"
    report_content += "The quick benchmark tests provide a snapshot of the network performance across selected servers. The average bandwidths indicate the network capacity under different parallel connections.\n"

    # Write the complete report to the markdown file
    with open(REPORT_FILE, "w") as f:
        f.write(report_content)

if __name__ == "__main__":
    main()
```
## Benchmark Results
```markdown
# Quick Benchmark Detailed Report

**Test Date:** 2024-11-15 00:47:03

## Introduction

This report summarizes the quick iPerf network performance tests conducted on selected servers. The tests measure bandwidth for IPv4 and IPv6 connections under various parallel connections.

## Methodology

The tests were performed using the iPerf3 tool with the following configurations:

- **Interval:** 0.3 seconds
- **Maximum Parallel Connections:** 6
- **Servers Tested:**
  - bhs.proof.ovh.ca -p 5201-5210
  - speedtest.nyc1.us.leaseweb.net -p 5201-5210
  - speedtest.chi11.us.leaseweb.net -p 5201-5210

Each server was tested under the following scenarios:

- Basic IPv4 and IPv6 tests
- Parallel connections with 2, 4, and 6 streams

## Summary of Results

| Server | IPv4 Average Bandwidth (Mbits/sec) | IPv6 Average Bandwidth (Mbits/sec) |
|--------|---------------------------|---------------------------|
| bhs.proof.ovh.ca | 14.10 | 13.50 |
| speedtest.nyc1.us.leaseweb.net | 14.78 | 12.14 |
| speedtest.chi11.us.leaseweb.net | 14.50 | 15.02 |

## Detailed Test Results

### Results for bhs.proof.ovh.ca

| Test | Bandwidth | Jitter | Packet Loss |
|------|-----------|--------|-------------|
| basic_ipv4 | 12.9 Mbits/sec | N/A | N/A |
| basic_ipv6 | 13.5 Mbits/sec | N/A | N/A |
| parallel_ipv4_2 | 15.3 Mbits/sec | N/A | N/A |
| parallel_ipv6_2 | N/A | N/A | N/A |
| parallel_ipv4_4 | N/A | N/A | N/A |
| parallel_ipv6_4 | N/A | N/A | N/A |
| parallel_ipv4_6 | N/A | N/A | N/A |
| parallel_ipv6_6 | N/A | N/A | N/A |

### Results for speedtest.nyc1.us.leaseweb.net

| Test | Bandwidth | Jitter | Packet Loss |
|------|-----------|--------|-------------|
| basic_ipv4 | 14.4 Mbits/sec | N/A | N/A |
| basic_ipv6 | 6.65 Mbits/sec | N/A | N/A |
| parallel_ipv4_2 | 15.3 Mbits/sec | N/A | N/A |
| parallel_ipv6_2 | 14.5 Mbits/sec | N/A | N/A |
| parallel_ipv4_4 | 14.5 Mbits/sec | N/A | N/A |
| parallel_ipv6_4 | 12.4 Mbits/sec | N/A | N/A |
| parallel_ipv4_6 | 14.9 Mbits/sec | N/A | N/A |
| parallel_ipv6_6 | 15.0 Mbits/sec | N/A | N/A |

### Results for speedtest.chi11.us.leaseweb.net

| Test | Bandwidth | Jitter | Packet Loss |
|------|-----------|--------|-------------|
| basic_ipv4 | 15.1 Mbits/sec | N/A | N/A |
| basic_ipv6 | 15.6 Mbits/sec | N/A | N/A |
| parallel_ipv4_2 | 15.2 Mbits/sec | N/A | N/A |
| parallel_ipv6_2 | 15.7 Mbits/sec | N/A | N/A |
| parallel_ipv4_4 | 14.3 Mbits/sec | N/A | N/A |
| parallel_ipv6_4 | 13.5 Mbits/sec | N/A | N/A |
| parallel_ipv4_6 | 13.4 Mbits/sec | N/A | N/A |
| parallel_ipv6_6 | 15.3 Mbits/sec | N/A | N/A |

## Conclusion

The quick benchmark tests provide a snapshot of the network performance across selected servers. The average bandwidths indicate the network capacity under different parallel connections.
```

## Future Work
- **Auto-Updating Server List:** Implementing a mechanism to automatically fetch and update the list of available iPerf servers ensures tests are always current.
- **Measuring Latency and DNS Resolution Times:** Incorporating latency measurements and DNS lookup times provides a more comprehensive view of network performance.
- **Analyzing Jitter and Packet Loss:** Beyond bandwidth, factors like jitter and packet loss significantly impact network performance, especially for real-time applications.
- **Implementing Error Handling and Retries:** Enhancing the script with robust error handling and retry mechanisms improves reliability.
- **Support for Additional Protocols:** Including tests for UDP and other protocols can provide a more complete performance analysis.
- **Advanced Statistical Analysis:** Calculating statistics like variance and standard deviation can offer deeper insights into network performance variability.
---
