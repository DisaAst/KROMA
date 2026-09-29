# Working with features and data to maximize scores

## Main conclusion

According to the current IoU logo, most likely go in order `[фон, класс 1, класс 2, класс 3]`:

- background: `0.8847`;
- weak grade, class 1: `0.3062`;
- middle degree, class 2: `0.5054`;
- strong degree, class 3: `0.6671`.

`mIoU=0.5909` includes the background and therefore looks optimistic. Checkpoint is saved by the correct target metric:

```text
(0.3062 + 0.5054 + 0.6671) / 3 = 0.4929
```

The main problem of class 1 is not an ordinary imbalance. In the training sample, the distribution of classes is `40.0 / 37.1 / 22.9%`, that is, the weak degree is the most frequent class. It sags because it is spectrally closest to the background, often lies along the edge of the burn and depends on the type of cover.

It follows directly from [case-making](../description/Постановка_кейса_—_Мониторинг_природных_пожаров_КосмоХакатон_2026.pdf): The degree of damage is relative to the type of vegetation, and the train and private test are separated simultaneously in time and territory.

Official metric:

```text
Score = 0.35 × F1_af + 0.35 × IoU_burn + 0.30 × mIoU_sev
```

Therefore, improving `IoU_1` from `0.306` to `0.45` will give approximately `+0.0144` of the final Score only through the severity component, and while improving the burn contour – even more. The metric is described in more detail in [evaluation criteria](../description/Критерии_оценки_—_Мониторинг_природных_пожаров_КосмоХакатон_2026.pdf).

## 1. Fix validation first

It's more important than changing architecture.

### Breakdown

You can not do the usual random split on chips or patches:

- First divide the chips by `fire_event_id`;
- Only after that, cut the patches;
- make a separate time-shift validation: for example, to study for 2019-2023 and be checked for 2024;
- If the geography of the train is available, in addition, leave the entire region out of training.

The private test is all about 2025, so random validation almost certainly overstates the expected private score.

### Metrics

For the BS-module, you should save the model for:

```text
BSScore = 0.35 × IoU_burn + 0.30 × mIoU_sev
```

At the same time:

- `burn = target > 0`;
- `mIoU_sev` is considered only in classes 1-3;
- TP, FP and FN are summed up across all validation chips rather than being averaged across images.

Do not focus on the common F1 or mIoU with the background. `F1-score=0.719...` from the current log cannot be matched to the official `F1_af` until it is not known exactly how it is calculated.

## 2. Discover why Class 1 is lost

You need to build a confusion matrix and divide class 1 errors into several groups.

| Mistake | Probable cause | What do you do? |
| --- | --- | --- |
| `1 → 0` | Weak smoke looks like a background | Separate burn-head, oversampling weak borders, landcover-aware dNBR |
| `0 → 1` | Arable land, shadows, phenology | SCL, hard negatives, NDVI/NDMI, more stringent burn threshold |
| `1 → 2` | One threshold for different covers | Landcover and distances to landcover-specific thresholds |
| `2 → 1` | The model is not sensitive enough to the magnitude of the change | Ordinal loss, dNBR/RdNBR, signed band deltas |

Precision and recall of class 1 must be calculated:

- Low recall - the model does not see a weak burn;
- Low precision – the model paints in class 1 background and agricultural changes;
- Both indicators are acceptable, but IoU is low - many errors on the boundaries.

It is useful to make slices of IoU by:

- land-cover;
- `cloud_frac`;
- Time interval between `pre` and `post`;
- year and season;
- site;
- boundary and interior;
- dNBR.

## 3. Processing of input data

### Sentinel-2

B2–B12 channels come as `uint16` on a `0–10000` scale:

```python
reflectance = raw.astype("float32") / 10000.0
reflectance = np.clip(reflectance, 0.0, 1.2)
```

Normalization of `mean/std` should be considered only on the train part of the current fold. It is not necessary to do independent normalization of each chip: it can remove absolute differences between dates.

SCL is not a numerical continuous channel. It should be:

- classify;
- Convert to one-hot or embedding;
- Separately display the masks `cloud`, `shadow`, `water`, `snow`, `invalid`;
- Use only the nearest-neighbor.

### Sentinel-1

VV/VH values have the `×100` scale:

```python
sar_db = raw.astype("float32") / 100.0
sar_db = np.clip(sar_db, -35.0, 5.0)
```

You can then use the train mean/std. Both absolute values and `post - pre` differences are useful.

### Passes

For each data source, you need:

1. Create a mask of validity.
2. After normalization, fill in invalid values with zero.
3. Transmit the validity mask to the model by a separate channel.

It is impossible to simply replace NaN with zero without a mask channel: the model will assume the absence of observation for the real value.

### Metadata

It can be used as signs:

- `valid_frac`;
- `cloud_frac`.

Only split, EDA and quality control should be used:

- date;
- `fire_event_id`;
- `sev1_px`, `sev2_px`, `sev3_px`;
- `burn_area_ha`.

The last fields are missing from the test or are a direct target leak.

## 4. Signs for BS

Start with a compact set, without adding dozens of correlated indices at once.

### Spectral indices

For each date:

```text
NBR  = (B8A - B12) / (B8A + B12 + ε)
NDVI = (B8A - B4)  / (B8A + B4  + ε)
NDMI = (B8A - B11) / (B8A + B11 + ε)
```

Between dates:

```text
dNBR  = NBR_pre  - NBR_post
dNDVI = NDVI_pre - NDVI_post
dNDMI = NDMI_pre - NDMI_post
RdNBR = dNBR / sqrt(abs(NBR_pre) + ε)
```

`RdNBR` must be limited to a reasonable range, otherwise the `NBR_pre ≈ 0` values will become abnormally large.

The model should be transferred simultaneously:

- reference `pre`;
- reference `post`;
- `post - pre`;
- `abs(post - pre)`;
- indexes;
- indexing.

Signed difference shows the direction of change, absolute difference.

### The Most Important Feature for Class 1

In the reference markup, different dNBR boundaries were used for different covers:

| Cover | DNBR boundaries |
| --- | --- |
| Forest/shrubland | `0.10 / 0.27 / 0.66` |
| Grass cover | `0.062 / 0.204 / 0.386` |
| arable | `0.07 / 0.17 / 0.38` |
| Swamp/plainland | `0.075 / 0.331 / 0.677` |

Therefore, it is useful to add three channels of distances to thresholds:

```python
thresholds = threshold_lookup[landcover_group]  # H × W × 3
threshold_distance = dnbr[..., None] - thresholds
```

This is more useful than simply transmitting dNBR: the model immediately sees how much the pixel is above or below the boundary of a particular class for a given cover.

These values can be used as:

- supplementary;
- prior to severity logits;
- independent baseline for comparison.

Do not turn the thresholds into a final hard decision: phenology, arable land, clouds and smoke create false changes.

### Sentinel-1

It should be added:

- `VV_pre`, `VH_pre`;
- `VV_post`, `VH_post`;
- `ΔVV`, `ΔVH`;
- `VH - VV` for both dates.

For the forest, a change in VH is especially useful, and for steppe and arable land it can be more informative than VV. SAR is better used as a confirmation of optics, rather than as the only source of contour.

### Relief and cover

- elevation - standardize;
- Slope – Lead to a stable range;
- aspect - coded as `sin(aspect)` and `cos(aspect)`;
- landcover - one-hot or embedding;
- additionally create enlarged categories: forest, grass, arable land, water, development, etc.

## 5. Separate contour and severity

One softmax model `0/1/2/3` causes a weak fumes at the same time:

1. Competing with the background.
2. Competing with Classes 2 and 3.

It's better to make two heads.

### Burn head

Binary segmentation:

```python
target_burn = target > 0
burn_logits = model.burn_head(features)
```

### Severity head

Classes 1–3 only inside a real burn:

```python
severity_target = target[target_burn] - 1
severity_loss = cross_entropy(
    severity_logits.permute(0, 2, 3, 1)[target_burn],
    severity_target,
)
```

### Decoding

```python
p_burn = torch.sigmoid(burn_logits)
severity_logits = severity_logits + class_bias

pred = torch.zeros_like(p_burn, dtype=torch.long)
inside = p_burn > burn_threshold
pred[inside] = severity_logits.argmax(dim=1)[inside] + 1
```

This decoding also ensures that classes in submission do not overlap.

A good start-up loss function:

```text
L =
  0.45 × (BCE_burn + SoftIoU_burn)
+ 0.45 × (CE_severity + Lovasz_severity)
+ 0.10 × OrdinalLoss
```

Classes of severity ordered: `1 < 2 < 3`. The optional ordinal-loss helps the model understand that the `1 → 2` error is less gross than `1 → 3`.

Do not start with large inverse-frequency weights. Class 1 is not rare, and class 3 is rare and already recognized better.

## 6. Sampling and augmentation

If 512×512 is in memory, it is advisable to learn on full chips at least part of the era. If you need crops, you can use this mixture:

- 40% - patches with the `фон ↔ класс 1` border;
- 25% – patches with `1 ↔ 2` class mixing;
- 20% - hard negatives: arable land, water, cloud shadows;
- 15% - random patches.

After the first training, you need to collect FP/FN models and conduct the second stage of hard-negative mining.

Safe augmentations:

- horizontal and vertical reflection;
- 90° cornering;
- small, consistent variations in brightness;
- low-noise.

The same geometric transformation must be applied to `pre`, `post`, SAR, auxiliary and mask.

Don't:

- Change `pre` and `post` in places;
- Use a strong RGB color jitter;
- Interpolate SCL/landcover bilinearly;
- Use strong elastic distortions;
- Change the spectral channels independently and strongly.

## 7. Selection of thresholds after training

Out-of-fold predictions need to be made:

- `burn_threshold`;
- bias in class 1;
- bias in class 2;
- bias in class 3.

Pooled metrics should be optimized:

```text
0.35 × IoU_burn + 0.30 × mIoU_sev
```

Example of search ranges:

```text
burn_threshold: 0.30 … 0.70
bias_class_1:  -0.5 … +0.5
bias_class_2:  -0.3 … +0.3
bias_class_3:  -0.3 … +0.3
```

If Class 1 has a low recall, positive bias can help. Low precision requires hard-negative mining and/or a more stringent burn threshold.

Aggressive removal of small components should not be used: a weak degree often forms a thin stripe along the edge of the burner, and the morphology of the first destroys it.

## 8. Recommended line of experiments

In each experiment, it is necessary to fix the same split, seed and metric.

1. `Raw pre/post + U-Net`.
2. `+ signed/absolute band differences`.
3. `+ NBR, dNBR, NDVI, NDMI`.
4. `+ landcover`.
5. `+ landcover-specific threshold distances`.
6. `+ SCL masks`.
7. `+ Sentinel-1`.
8. `+ DEM/slope/aspect`.
9. Two heads: burn + severity.
10. Ordinal loss.
11. Hard-negative mining.
12. OOF threshold/logit calibration.
13. Averaging 2-3 models or fold ensemble.

Maximum contribution for Class 1 is expected from paragraphs 5, 9, 11 and 12.

## 9. AF module

Even a very good severity gives only 30% of the total metric. For AF critical:

- `I4`;
- `I5`;
- `I4 - I5`;
- Local deviations of I4 and `I4 - I5` from the background in windows of different sizes;
- saturation4;
- I3 for solar glare;
- landcover;
- solar zenith and day/night;
- validity-mask.

Positive AF pixels of all `0.035%`, so a regular BCE on all pixels is almost guaranteed to teach the background. Need focal/Tversky loss, oversampling positive candidates and hard negatives.

## Practical plan for the next iterations

If time is short, the best course of action is:

1. Implement the correct `GroupKFold` and official evaluator.
2. Analyze `1 → 0` Confusion vs. `1 → 2`.
3. Add dNBR and landcover-specific threshold-distance features.
4. Divide the model into burn-head and severity-head.
5. Add oversampling to weak burn boundaries.
6. Select the burn threshold and class-1 bias by OOF.
7. Only then add Sentinel-1, new architectures and ensembles.

In the file ["Reference to data"](../description/Ссылка_на_данные.pdf) is only a reference to the issuance of data and the initial time of publication of the test; there is no additional information about the features in it.
