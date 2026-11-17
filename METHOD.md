# How the forecast works

## The shape of a launch day

A big console game unlocks at midnight local time, country by country. On
the UTC release day that means the first hour belongs mostly to New Zealand
and Australia, the count dips while Europe sleeps, then climbs as Europe and
the Americas unlock, peaking in the evening. The shape also depends strongly
on the launch's size: a small launch surges many times over its first hour, a
huge one barely grows at all. Across past launches the peak ratio falls
almost in inverse proportion to the hour-1 count (correlation about -0.94 in
log space), so the peak hour's count is nearly the same whatever the launch's
size. The forecast is therefore anchored on the case's own hour-1 print and
borrows the shape from past launches in two ways, blended evenly.

## The model

1. **Pool.** A past launch counts when its hourly series covers the whole
   release day and its hour-1 count is at least 100 viewers. Launches the
   case flags as weak analogues or early-access contaminated are left out
   (early access inflates their hour-1 counts, which flattens their shape);
   if fewer than 13 clean launches remain, the flagged ones come back in.
2. **Neighbours.** The 8 comparables whose hour-1 count is closest to
   this case's hour-1 print, in log space. For each hour, the log-ratio
   log(V(h) / V(1)) across those 8 has a mean (the median log-ratio) and a
   standard deviation (the spread). The forecast is log-normal around that
   mean: q = p1 * exp(mu + z * sd), with z the standard normal quantiles at
   0.05, 0.25, 0.5, 0.75 and 0.95.
3. **Size regression.** For each hour, a weighted linear regression of the
   log-ratio on log hour-1 count across the usable pool, with Gaussian
   weights that favour launches near this case's size (bandwidth 1.5 in log
   space). This is the part that extrapolates the size trend: for a launch
   bigger than any past one it predicts a flatter day than the neighbours
   alone would, because the trend says the bigger the launch, the flatter
   the day. Its spread is the weighted residual spread around the line.
   Fitting the line on all launches at once would be wrong: the small
   launches' extreme ratios would drag it down and make it predict an
   evening collapse for a huge launch. The Gaussian weights keep the fit
   local to the launch's own scale.
4. **Blend.** The two models' quantiles are blended evenly in log space
   (their geometric mean). The spread of each is shrunk by 0.85, because a
   pool of launches with different unlock patterns overstates the
   dispersion of any one launch, and floored at 0.30 log units so the band
   never collapses.
5. **Fallback.** With no usable comparable at all, the forecast is
   persistence: every hour stays at the hour-1 print with a widening
   log-normal spread.

## Validation

Scored against the SullyGnome hourly counts of the four practice launches
(Marvel's Wolverine, Red Dead Redemption 2, God of War Ragnarok, Marvel's
Spider-Man 2), the model reaches a mean pinball loss of 0.16, against 0.39
for the house baseline and 0.18 for the neighbours-only model it replaces.
The gain is not fitted to those four cases: in leave-one-out over the big
past launches the blend scores about 0.26 against 0.29 for the
neighbours-only model, and over the biggest launches (hour-1 above 100,000
viewers, the launches most like GTA VI) about 0.20 against 0.25. The same
ranking holds when the four practice launches are added to the pool, as the
final's case file does. The data is the case file's own comparables, so the
program needs no network and no files beyond its case file. It is
deterministic: a pure function of the case file, so two runs write the same
bytes.
