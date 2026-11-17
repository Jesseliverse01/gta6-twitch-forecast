# How the forecast works

## The shape of a launch day

A big console game unlocks at midnight local time, country by country. On
the UTC release day that means the first hour belongs mostly to New Zealand
and Australia, the count dips while Europe sleeps, then climbs as Europe and
the Americas unlock, peaking in the evening. The shape also scales with the
launch: a small launch surges many times over its first hour, a huge one
barely doubles. So the forecast is anchored on the case's own hour-1 print
and borrows the shape from past launches of a similar size.

## The model

1. **Pool.** A past launch counts when its hourly series covers the whole
   release day and its hour-1 count is at least 100 viewers. Launches the
   case flags as weak analogues or early-access contaminated are left out
   (early access inflates their hour-1 counts, which flattens their shape);
   if fewer than 13 clean launches remain, the flagged ones come back in.
2. **Neighbours.** The 13 comparables whose hour-1 count is closest to
   this case's hour-1 print, in log space.
3. **Quantiles.** For each hour, the log-ratio log(V(h) / V(1)) across those
   13 launches has a mean (the median log-ratio) and a standard deviation
   (the spread). The forecast is log-normal around that mean:
   q = p1 * exp(mu + z * sd), with z the standard normal quantiles at
   0.05, 0.25, 0.5, 0.75 and 0.95. The spread is shrunk by 0.85, because a
   pool of launches with different unlock patterns overstates the
   dispersion of any one launch, and floored at 0.30 log units.
4. **Fallback.** With no usable comparable at all, the forecast is
   persistence: every hour stays at the hour-1 print with a widening
   log-normal spread.

## Validation

Scored against the SullyGnome hourly counts of the four practice launches
(Marvel's Wolverine, Red Dead Redemption 2, God of War Ragnarok, Marvel's
Spider-Man 2), the model reaches a mean pinball loss of 0.18, against 0.39
for the house baseline. The same ranking holds in leave-one-out over the
big past launches. The data is the case file's own comparables, so the
program needs no network and no files beyond its case file. It is
deterministic: a pure function of the case file, so two runs write the same
bytes.
