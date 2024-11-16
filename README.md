# Spotlight

This repository contains the schema and links to the dataset for **Anomaly Detection in OpenRAN**. The data was collected from an **Enterprise Grade Indoor 5G Testbed** using a Capgemini vDU, vCU, and Intel FlexRAN L1.

bibtex here.
## Directory Structure

### `schema/`

- **`radio_kpi.md`**  
  This file contains the schema for the Radio KPI. It provides a detailed explanation of each KPI, including whether they are derived from other KPIs.

- **`platform_kpi.md`**  
  This file contains the schema for the Platform KPI. The schema in this file applies to all RAN and PTP threads.

- **`network_kpi.md`**  
  This file contains the schema for the Network KPI. The schema applies to the ports on the switch used by DUs and RUs.

### `sample/`

This folder contains:  
- **Sample Radio and Platform KPI** files for one anomaly, including the corresponding labels file.  
- **One Raw KPI Sample** to illustrate the how raw data collected.

## Dataset Info
Download [Dataset](https://drive.google.com/drive/folders/1x7WnU6q9EodacUdh3iXRmi9FWrMD3rME?usp=sharing)

We provide the **baseline and anomalous datasets with label** for anomaly detection.

This data was collected under the following traffic scenarios:  
- **1 UE** and **5 UEs** generating TCP, UDP, video, file download, ping, and HTTP traffic.  
- **7 UEs** performing ping traffic.

We created anomalies in the following components:  
- **Platform Anomalies** targeting the PDCP layer and MAC scheduler.  
- **Network Anomalies** targeting the FH link.  
- **Radio Anomalies** targeting the PHY layer.
- **Mixed Anomalies** Any possible combination of above anomalies.

## Anomaly Creation

### Methodology:
Artificial anomalies are created by mimicking real-world anomaly scenarios. These scenarios include:

- **CPU Contention:**  
  CPU contention occurs when multiple processes or applications compete for the same CPU resources, leading to performance degradation. This can be triggered in two ways:
  - **External Process:** External processes running on the system can cause CPU contention.
  - **Misplacement of RAN Threads:** The placement of RAN threads on CPU cores where they should not run can also lead to contention.

In our experiments, three anomalies are caused by CPU contention:

1. **CU Contention:**  
   To induce CU contention, the `pdcp_worker_0` thread is subjected to stress using the tool **stress-ng**. The stress is applied periodically with varying durations to create different anomaly mixes.

2. **Scheduler Contention:**  
   To induce scheduler contention, the `TxCtrl_DU11_C0` thread is stressed by running a program in a while loop performing some dummy operation. (*Note: RAN can crash if we use stress-ng to stress the scheduler thread.*)

3. **Thread Misconfiguration:**  
   To induce thread misconfiguration and CPU contention, we modify the thread affinity using the **taskset** command. This involves setting a thread to run on a CPU core where it is not intended to execute.

### External Interference:

- **Network Contention:**  
  Network contention arises when other radios, such as neighboring cells or external interference sources, operate on the same carrier frequency as the RAN (Radio Access Network). In our setup, we use **USRP** operating in the same frequency to create external interference.

- **FH Network Contention:**  
  Network contention can occur on a link responsible for transmitting fronthaul traffic.

## Appendix

### Procedure-1: (Used to Calculate Total Runtime from Exponential Histogram)

In an **exponential histogram**, the bucket size is not fixed. Instead, the bucket size follows a pattern where it is defined as **2^i**, where **i = 0…n**.

```python
def calc_runtime(self, bin): 
    total = 0 
    for i, bin_count in enumerate(bin): 
        if bin_count > 0: 
            range_start = 2 ** i 
            range_end = 2 ** (i + 1) - 1 
            range_size = range_end - range_start + 1 
            items_per_bin = bin_count / range_size 
            # Calculate total for this bin using formulas 
            total += items_per_bin * ((range_start + range_end) * range_size / 2) 
    return total
```
# Procedure-2: Calculate Average/Mean from Shift Type Histogram

In a shift-type histogram, the bucket size is not fixed as in an exponential histogram. Instead, the **shift** value is used to calculate the size of each bucket. The formula for the bucket size is:

\[
\text{Bucket Size} = 2^{\text{shift}}
\]

This means each bucket can hold \( 2^\text{shift} \) elements. Different shifts are used depending on the range of the KPI being measured. For example:
- For **MCS (Modulation and Coding Scheme)**, the shift is 2.
- For **PRB (Physical Resource Block)**, the shift is 5.

## Procedure 2 : Calculate Average/Mean

The procedure to calculate the average/mean from a shift-type histogram involves the following steps:

```python
def calc_avg(self, bin, shift, min, max): 
    total = 0 
    BUCKET = 2 ** shift 
    hist_value = [0] * 9 
    cnt = 0 
    
    for i, bin_count in enumerate(bin): 
        if bin_count > 0: 
            cnt += 1 

            # Determine the range for the current bin
            if BUCKET * i <= min: 
                range_start = min 
            else: 
                range_start = BUCKET * i 

            if BUCKET * (i + 1) - 1 > max: 
                range_end = max 
            else: 
                range_end = BUCKET * (i + 1) - 1 

            # Calculate the size of the range and the items per bin
            range_size = range_end - range_start + 1 
            items_per_bin = bin_count / range_size 

            # Calculate total for this bin using the formula
            total += ((range_start + range_end) * range_size / 2) * items_per_bin 

    # Return the average by dividing total by the sum of all bin counts
    return total / sum(bin)

