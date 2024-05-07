# Underwater Test
## Analysis_PMT_uw.ipynb
In the underwater test, since we want to measure the dark rates of PMTs, we need to set the ADC to be self-triggered and the threshold to 20 ADC. To minimize the statistical uncertainty and cancel out the fluctuation, the DAQ time is chosen to be 10 mins, which will generate around 20 GB of data. In this case, we have to cut the data into several smaller chunks (epochs) to analyze. We also use the timestamps of the triggered events to reconstruct the time of all peaks we found, to monitor the dark rate across the DAQ period.

Here are the comments for codes:
```
PMT_num = 1107
bins = 1000
range_uw = 1000
epoch_size = 10000000
Threshold = 40
```

``PMT_num``, ``bins`` and ``range_uw`` are similiar to the definitions in ``Analysis_PMT_darkbox.ipynb``. ``epoch_size`` is the size of smaller chunk of data, and ``Threshold`` is the threshold for peak-finding function.


```
def cut_file_in_epoch(file_path, outputpath, epoch_size):
    with open(file_path, 'r') as file:
        base_name = file_path.split('.')[0]
        num = 0
        size = 0
        timestamp = 0
        epoch = []
        time_epoch = []

        int_limit = 4294967294
        int_back = 2147483647
        repitition = 0
        time_temp = 0
        time_prev = 0
        for line in file:
            if size < epoch_size:
                if 'Trigger' in line:
                    time_temp = int(line.split(': ')[1].strip())
                    if time_temp < int_back:
                        timestamp = time_temp
                    else:
                        if time_prev - time_temp > 100000:
                            repitition += 1
                        timestamp = repitition * (int_limit - int_back + 1) + time_temp
                    time_prev = time_temp
                if ('Record Length' not in line) and ('BoardID' not in line) and ('Channel' not in line) and ('Event Number' not in line) and ('Pattern' not in line) and ('Trigger Time Stamp' not in line) and ('DC offset' not in line):
                    epoch.append(line)
                    time_epoch.append(str(timestamp)+'\n')
                    timestamp = timestamp + 1
                    size += 1
            else:
                num = num + 1
                size = 0
                epoch_name = outputpath + 'PMT' + str(PMT_num) + '_epoch' + str(num) + '.txt'
                time_stamp_name = outputpath + 'PMT' + str(PMT_num) + '_timestamp_epoch' + str(num) + '.txt'
                with open(epoch_name, 'w') as epoch_file:
                    epoch_file.writelines(epoch)
                print('Epoch file ' + epoch_name + ' has been created.')
                with open(time_stamp_name, 'w') as time_stamp_file:
                    time_stamp_file.writelines(time_epoch)
                print('Timestamp file ' + time_stamp_name + ' has been created.')
                epoch = []
                time_epoch = []
        #if line reaches the end of the file
        if epoch:
            num = num + 1
            epoch_name = outputpath + 'PMT' + str(PMT_num) + '_epoch' + str(num) + '.txt'
            time_stamp_name = outputpath + 'PMT' + str(PMT_num) + '_timestamp_epoch' + str(num) + '.txt'
            with open(epoch_name, 'w') as epoch_file:
                epoch_file.writelines(epoch)
            print('Epoch file ' + epoch_name + ' has been created.')
            with open(time_stamp_name, 'w') as time_stamp_file:
                time_stamp_file.writelines(time_epoch)
            print('Timestamp file ' + time_stamp_name + ' has been created.')
            epoch = []
            time_epoch = []

    print('All epochs have been created.')
    print('Total number of epochs: ' + str(num))
    return num
```
1. Opens a file at the given file_path and reads it line by line.

2. It checks if the current line contains certain keywords. If it contains 'Trigger', it extracts a timestamp. If it doesn't contain any of the specified keywords, it appends the line to the epoch list and the timestamp to the time_epoch list.

3. It keeps track of the number of lines added to the epoch list. Once this number reaches epoch_size, it writes the epoch list to a new file and the time_epoch list to another new file. The new files are named based on the PMT_num and the current epoch number.

4. It then resets the epoch and time_epoch lists and continues with the next set of lines from the input file.

5. If it reaches the end of the input file and there are still lines in the epoch list, it writes these to a new file as well.

6. Finally, it prints the total number of epochs created and returns this number.


This function is designed to divide a large file into smaller segments, referred to as ``epochs``, with each ``epoch`` comprising a specified number of lines. Additionally, it assigns a timestamp to each line. The function also addresses the scenario where timestamps exceed a maximum value, ``int_limit``, and reset to a predefined value, ``int_back``. This behavior typically occurs due to the rollover mechanism in our Analog-to-Digital Converter (ADC), the DT5720, which I presume uses a 32-bit integer to store time values.

The rest of the codes are either very similar to ``Analysis_PMT_darkbox.ipynb`` or fairly straightforward.




