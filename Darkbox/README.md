# Dark Box Test
## Analysis_PMT_darkbox.ipynb
``Analysis_PMT_darkbox.ipynb`` is the script to directly analyze the data named with ``PMT%d_lighton_%d.dat`` or ``PMT%d_dark_%d.dat``, which means the ``PMT%d`` in ``lighton`` or ``dark`` at ``%d`` Volts. The script generates histograms for the pulse areas in both light-on and dark conditions. It also provides information on the gain vs voltage, as well as the photon count for each voltage.

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
light_on = 0: This line is used to track whether a light is on or off, with 0 indicating 'off', which means ``PMT%d_dark_%d.dat``, and 1 indicating 'on', which means ``PMT%d_lighton_%d.dat``. 

PMT_num = 1107: This line assigns the value for ``PMT%d`` and further sets the PMT number for titles of plots.

testing = 0: This flag indicates whether the program is in testing mode. In testing mode, only 30% of the data is used to save time and memory.

voltage_use = [1400, 1500, 1600, 1700, 1800, 1900]: This line is creating a variable named voltage_use and assigning it a list of integers. These values represent different voltage levels to be used in an experiment or test.

bins_lighton = 100 and bins_dark = 50: Set the bins for histograms, for the histogram of light-on or dark respectively.

range_lighton = 12000 and range_dark = 600: Set the maximum range for histograms (0~range_*), for the histogram of light-on or dark respectively.
