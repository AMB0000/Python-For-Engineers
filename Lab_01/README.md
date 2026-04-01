# Lab 1 Setup Lab

## Part 1 — Environment Verification

Ran `sanity_check.py` successfully. Plot generated below.

<img src="https://raw.githubusercontent.com/AMB0000/Python-For-Engineers/77ada15b0430e6c7e8b95094ed1ef74b8fa860a5/Lab_01/Screenshots/Screenshot%202026-04-01%20082636.png" width="600">---

## Part 2 — Modified Signal

Edited the script to use frequency = 2 and noise level = 0.5:

```python
y = np.sin(2*x) + 0.5*np.random.randn(len(x))
```

![Part 2 Plot](https://raw.githubusercontent.com/AMB0000/Python-For-Engineers/2d85b9efcba032fc4f0d499ccc1eea2cbdc3d1c8/Lab_01/Screenshots/part2_notitle.png)
---

## Reflection

Noise adds random fluctuations to the signal, making it harder to identify the clean underlying waveform. Increasing the frequency compresses the sine wave, producing more cycles over the same x-range. High noise combined with high frequency would make the data very difficult to analyze since individual cycles become indistinguishable from the random variation. Filtering or averaging techniques would be needed to recover the true signal in that case.
