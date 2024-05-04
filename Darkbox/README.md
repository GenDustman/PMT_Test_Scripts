# Dark Box Test
## Analysis_PMT_darkbox.ipynb
``Analysis_PMT_darkbox.ipynb`` is the script to directly analyze the data named with ``PMT%d_lighton_%d.dat`` or ``PMT%d_dark_%d.dat``, which means the ``PMT%d`` in ``lighton`` or ``dark`` at ``%d`` Volts. The script generates histograms for the pulse areas in both light-on and dark conditions. It also provides information on the gain vs voltage, as well as the photon count for each voltage. To make future analysis easier, it will also generate ``PMT%d_lighton_%d_peaks_area.txt`` or ``PMT%d_dark_%d_peaks_area.txt`` which contains the areas of all peaks it has found in data.

Here are the comments for the codes:

```
light_on = 0
PMT_num = 1107
testing = 0
voltage_use = [1400, 1500, 1600, 1700, 1800, 1900]
bins_lighton = 100
bins_dark = 50
range_lighton = 12000
range_dark = 600
```
``light_on = 0``: This line is used to track whether a light is on or off, with 0 indicating 'off', which means ``PMT%d_dark_%d.dat``, and 1 indicating 'on', which means ``PMT%d_lighton_%d.dat``. 

``PMT_num = 1107``: This line assigns the value for ``PMT%d`` and further sets the PMT number for titles of plots.

``testing = 0``: This flag indicates whether the program is in testing mode. In testing mode, only 30% of the data is used to save time and memory.

``voltage_use = [1400, 1500, 1600, 1700, 1800, 1900]``: This line is creating a variable named voltage_use and assigning it a list of integers. These values represent different voltage levels to be used in an experiment or test.

``bins_lighton = 100`` and ``bins_dark = 50``: Set the bins for histograms, for the histogram of light-on or dark respectively.

``range_lighton = 12000`` and ``range_dark = 600``: Set the maximum range for histograms (0~range_*), for the histogram of light-on or dark respectively.

```
data = []
if light_on:
    for i in voltage_use:
        file_path = '/raid13/genli/Coherent/PMT_Testing/Testing_%d/PMT%d_lighton_%d.dat' % (PMT_num, PMT_num, i)
        print(file_path)
        data.append(np.fromfile(file_path, dtype=np.int16))
else:
    for i in voltage_use:
        file_path = '/raid13/genli/Coherent/PMT_Testing/Testing_%d/PMT%d_dark_%d.dat' % (PMT_num, PMT_num, i)
        print(file_path)
        data.append(np.fromfile(file_path, dtype=np.int16))
```
Choose the correct ``file_path`` for data. The data read from the files is then appended to the ``data`` list.

```
def get_baseline(data):
    #get median of first 10000 samples
    baseline = np.median(data[0:10000])
    return baseline

def flip_data(data):
    # flip data
    baseline = get_baseline(data)
    data = baseline - data
    return data
```
``flip_data`` code is used to flip a signal around a baseline value.

```
for i in range(len(data)):
    data[i] = flip_data(data[i])
```
All data are flipped and centered at ``baseline = 0``.

For functions ``def find_peaks_lighton(data)`` and ``def find_peaks_dark(data)``, they work very similarly in identifying peaks and calculating their areas, but with some small distinctions referring to the parameters used. So here we will only explain one of them:

```
def find_peaks_lighton(data):
    # find the threshold
    threshold = np.median(np.sort(data[0:int(len(data)*0.2)])[-int(len(data)*0.0002):])
    print('Threshold = ', threshold)
    # find peaks above threshold and higher than the previous and next data points
    peaks = np.where((data > threshold) & (data >= np.roll(data, 1)) & (data >= np.roll(data, -1)) & (data >= np.roll(data, 2)) & (data >= np.roll(data, -2)) & (data >= np.roll(data, 3)) & (data >= np.roll(data, -3)) & (data >= np.roll(data, 4)) & (data >= np.roll(data, -4))& (data >= np.roll(data, 5)) & (data >= np.roll(data, -5)))[0]
    #distance between peaks should be larger than 100
    peaks = peaks[np.where(np.diff(peaks) > 100)[0]]
    # find the start and the end of the peaks
    threshold_width = 0.10*data[peaks]
    start = np.zeros(len(peaks), dtype=int)
    end = np.zeros(len(peaks), dtype=int)
    for i in range(len(peaks)):
        start_t = peaks[i]
        end_t = peaks[i]
        while data[start_t] > threshold_width[i]:
            start_t -= 1
        start[i] = start_t
        while data[end_t] > threshold_width[i]:
            end_t += 1
        end[i] = end_t
    # find the peak height
    height = data[peaks]
    # find the peak width
    width = end - start
    # find the peak area
    area = np.zeros(len(start))
    for i in range(len(start)):
        area[i] = np.sum(data[start[i]:end[i]])
    return peaks, start, end, height, width, area

```

1. It first calculates a threshold value, which is the median of the top 0.02% values in the first 20% of the sorted data.

2. It then identifies peaks in the data. A peak is defined as a data point that is greater than the threshold and also greater than or equal to its immediate five neighbors on either side.

3. It further refines the peaks by ensuring that the distance between consecutive peaks is more than 100.

4. It then identifies the start and end of each peak. The start and end of a peak are defined as the points where the data value drops below 10% of the peak value.

5. It calculates the height of each peak, which is simply the data value at the peak.

6. It calculates the width of each peak, which is the difference between the end and start of the peak.

7. It calculates the area under each peak, which is the sum of the data values between the start and end of the peak.

8. Finally, it returns the peaks, start, end, height, width, and area of each peak.

``threshold`` in ``def find_peaks_lighton(data)`` can be chosen easily since the peaks (signal for multi-photoelectron in light-on test) are significant, however in the case of ``def find_peaks_dark(data)``, peaks for single-photoelectron are subtle and can vary a lot between different PMTs. This is why we designed the function to be "self-adaptive". Instead of using a fixed value, we use the median of a certain percentage of data. ``0.000007`` in ``def find_peaks_dark(data)`` is a crucial parameter and it is chosen based on both reasonable estimation and also experience. The dark rate is $~1000Hz$, and the width of peaks is around $5 ADC = 4\times 5 ns = 20 ns$, so the total time length of peaks is $2\times 10^-5 s~10^-5s$. Starting from $0.00001$, after some finetuning, we can finally get ``0.000007``, which can maximally pick all the peaks while rejecting the noises.





