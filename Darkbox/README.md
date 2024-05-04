# Dark Box Test
## Analysis_PMT_darkbox.ipynb
``Analysis_PMT_darkbox.ipynb`` is the script to directly analyze the data named with ``PMT%d_lighton_%d.dat`` or ``PMT%d_dark_%d.dat``, which means the ``PMT%d`` in ``lighton`` or ``dark`` at ``%d`` Volts. The script generates histograms for the pulse areas in both light-on and dark conditions. It also provides information on the gain vs voltage, as well as the photon count for each voltage.

```
ligh_on = 0
PMT_num = 1107
testing = 0
voltage_use = [1400, 1500, 1600, 1700, 1800, 1900]
bins_lighton = 100
bins_dark = 50
range_lighton = 12000
range_dark = 600
```

