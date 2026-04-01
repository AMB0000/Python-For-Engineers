# Lab 1 Setup Lab

## Part 1 — Environment Verification

Ran `sanity_check.py` successfully. Plot generated below.

![Part 1 Plot](part1_plot.png)

---

## Part 2 — Modified Signal

Edited the script to use frequency = 2 and noise level = 0.5:

```python
y = np.sin(2*x) + 0.5*np.random.randn(len(x))
```

![Part 2 Plot](part2_plot.png)

---

## Reflection

Noise adds random fluctuations to the signal, making it harder to identify the clean underlying waveform. Increasing the frequency compresses the sine wave, producing more cycles over the same x-range. High noise combined with high frequency would make the data very difficult to analyze since individual cycles become indistinguishable from the random variation. Filtering or averaging techniques would be needed to recover the true signal in that case.
