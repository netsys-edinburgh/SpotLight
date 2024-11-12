* Radio KPI
* Platform KPI
* Network KPI
Appendix: 

Procedure-1(Used to calculate total runtime form exponential histogram) 

In exponential histogram, bucket size is not fixed and bucket size is 2i where i=0…n  

 
    `def calc_runtime(self, bin): 

        total = 0 

        for i, bin_count in enumerate(bin): 

            if bin_count > 0: 

                range_start = 2 ** i 

                range_end = 2 ** (i + 1) - 1 

                range_size = range_end - range_start + 1 

                items_per_bin = bin_count / range_size 

                # Calculate total for this bin using formulas 

                total += items_per_bin * ((range_start + range_end) * range_size/2) 

 

        return total` 

 

Procedure-2 (Used to calculate avg/mean from shift type histogram ) 

In exponential histogram, bucket size is fixed. Shift is used to calculate size of bucket in histogram. Bucket size = 2shift i.e. each bucket can hold 2shift elements. Different shift is used based on the kpi range. E.g. for mcs shift is 2, for prb shift is 5 etc.  

    `def calc_avg(self, bin,shift, min, max): 

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

        return total/sum(bin)` 
