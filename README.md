* Radio KPI
* Platform KPI
* Network KPI
Anomaly Creation:  

Methodology: Artificial anomalies are created by mimicking real-world anomaly scenarios. These scenarios are: 

CPU contention:  CPU contention is when multiple processes or applications compete for the same CPU resources, leading to performance degradation. This can be triggered in two ways: by an external process or by the misplacement of RAN threads. 

External Process: External processes running on the system can cause CPU contention.   

Misplacement of RAN Threads: The placement of RAN threads on the CPU cores where they are not supposed to run can also lead to contention. 

In our experiments, three anomalies are caused by CPU contention: 

CU contention: To induce CU contention, the pdcp_worker_0 thread is subjected to stress using the tool stress-ng. The stress is applied periodically with varying durations to achieve different anomaly mixes. 

Scheduler contention: To induce Scheduler contention, the TxCtrl_DU11_C0 thread is stressed using program running while loop performing some dummy operation. (RAN can crash if we use stress-ng to stress the scheduler thread).  

Thread Misconfiguration: To induce thread misconfiguration and CPU contention, we modify the thread affinity by using the taskset command. This involves setting a thread to run on a CPU core where it is not intended to execute.   

  

External Interference: Network contention arises when other radios, such as neighboring cells or external interference sources, operate on the same carrier frequency as the RAN (Radio Access Network). We are using USRP operating in the same frequency to create External interference. 

FH Network Contention: Network contention occurs on a link responsible for transmitting fronthaul traffic. 
Appendix: 

Procedure-1(Used to calculate total runtime form exponential histogram) 

In exponential histogram, bucket size is not fixed and bucket size is 2i where i=0…n  

 ```
    def calc_runtime(self, bin): 

        total = 0 

        for i, bin_count in enumerate(bin): 

            if bin_count > 0: 

                range_start = 2 ** i 

                range_end = 2 ** (i + 1) - 1 

                range_size = range_end - range_start + 1 

                items_per_bin = bin_count / range_size 

                # Calculate total for this bin using formulas 

                total += items_per_bin * ((range_start + range_end) * range_size/2) 
                
        return total
```

 

Procedure-2 (Used to calculate avg/mean from shift type histogram ) 

In exponential histogram, bucket size is fixed. Shift is used to calculate size of bucket in histogram. Bucket size = 2shift i.e. each bucket can hold 2shift elements. Different shift is used based on the kpi range. E.g. for mcs shift is 2, for prb shift is 5 etc.  
```
    def calc_avg(self, bin,shift, min, max): 

        total = 0 

        BUCKET = 2**shift 

        hist_value = [0] * 9 

        cnt = 0 

        for i, bin_count in enumerate(bin): 

            if bin_count > 0: 

                cnt+=1 

                if(BUCKET*i<=min): 

                    range_start = min 

                else: 

                    range_start = BUCKET*i 

                if(BUCKET*(i + 1) - 1> max): 

                    range_end = max 

                else: 

                    range_end = BUCKET*(i + 1) - 1 

                range_size = range_end - range_start + 1 

                items_per_bin = bin_count / range_size 

                # Calculate total for this bin using formulas 

                total += ((range_start + range_end) * range_size / 2)*items_per_bin 

                # hist_value[i] = total 

                # total=0 

        return total/sum(bin)
``` 
