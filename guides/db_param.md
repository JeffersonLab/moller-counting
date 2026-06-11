# Explanations for .dat parameters

This contains a writeup of many of the parameters controlled in db_moller.***.dat files, with a corresponding explanation of its meaning.

## Decoding

Note, for info on common mode corrections, see: <https://misportal.jlab.org/sti/publications/16222/attachments/7664/1_Di_Danning_2019_PHD1.pdf>

Sorting: Sort the 128 APV channels, remove N low and N hi bins, then average and subtract the average (O(NlogN))
Danning: Compute mean and sigma, remove values 10 sigma from mean, then for niter repeat process (O(N)), more computationally efficient than above

| parameter                | Values           | Description |
| ------------------------ | ---------------- | ----------- |
| pedestalmode             | 0 or 1           | 1: compute pedestal and common mode corrections, 0: don't |
| commonmode_flag          | 0, 2, 3, 4       | Sorting, Histogramming, Danning (cm_min = 0), Danning respectively |
| commonmode_online_flag   | 0, 2, 3, 4       | Version of method above saved to histograms |
| commonmode_nstriplo      | 0 - 50           | Only used in sorting method, number of lowest bins removed after ADC values are sorted |
| commonmode_nstriphi      | 0 - 50           | Only used in sorting method, number of highest bins removed after ADC values are sorted |
| commonmode_niter         | 2 - 10           | Seems to only be used if commonmode_flag is 4, number of Danning iterations |
| commonmode_minstrips     | 1 - NChannels_APV| Zero suppressed we require reading at least minstrips+nstriplo+nstriphi |
| plot_common_mode         | 0 or 1           | Do we want common mode plots? I think it's intensive |
| commonmode_range_nsigma  |                  | cm_[min/max] is commonmode_mean(U/V) [-/+] commonmode_rms(U/V), seems to only be used if commonmode_flag = 4 |
| threshold_sample         |                  | min value of max ADC sample to keep strip (baseline-subtracted) |
| threshold_stripsum       |                  | min value of sum of ADC samples on a strip (baseline-subtracted) |
| threshold_clustersum     |                  | min value of sum of all ADCs over all strips in a cluster (baseline-subtracted) |
| corrcoeff_cut            | -1 < x < 1       | cut on U/V correlation coefficient (r value) defined as Covariance(U, V)/sigma_U*sigma_V |
| zerosuppress_nsigma      |                  | max adc value for a given strip must be above this * pedestal_mean |
| suppressfirstlast        |                  | 0: allow peaking in first or last sample 1: suppress peaking in first and last sample -1: suppress peaking in first sample only (or other negative number) -2: suppress peaking in last sample only: |
| [u/v]gain                |                  | Gain of U/V strips by APV card (ordered by strip position, NOT by order of appearance in decode map), can use this to center e.g. ADC asym |

## Clustering/2D hits

For filter1D, parameters being "cut" on are: threshold_clustersum, iff >0, also cut on nstrips >= 2
For filter2D, parameters being "cut" on are: U/V time difference, ADC asymmetry, and perhaps correlation coefficient.

| parameter                  | Values      | Description |
| -------------------------- | ----------- | ----------- |
| peakprominence_minsigma    |             | Peak prominence cut for separating clusters |
| peakprominence_minfraction |             | |
| sigmahitshape              |             | used in cluster splitting algorithm to compute the splitting fraction, maybe characteristic width of charge cluster?? |
| maxn[u/v]_charge           |             | n strips to use in each direction to calculate cluster charge |
| maxn[u/v]_pos              |             | n strips to use in each direction to calculate cluster position |
| filterflag1D               | -1, 0, 1    | -1: Don't filter 1D clusters, 0: filter iff one other cluster passed, 1: reject failing cluster no matter what |
| filterflag2D               | -1, 0, 1    | -1: Don't filter 2D hits, 0: filter iff one other hit passed, 1: reject failing hits no matter what |
| ADCasym_cut                | -1 < x < 1  | cut on 2D hits by ADC asymmetry, Abs(Asym) < cut |
| deltat_cut                 | In ns       | cut on 2D hits by U/V time difference |

## Tracking

The outline of the tracking algorithm is as follows:

1. Find all combinations of detectors s.t. nlayers > minlayers
2. Pair all possible free hits on the outermost detectors
3. Calculate a straight line between these two points
4. For intermediate layers, find all hits in grid bins pointed to by track
5. If each intermediate layer contains a hit in the grid bins, proceed
6. If "tryfasttrack" is true, find the hit closest to the track
7. If "tryfasttrack" is false, find all hit combinations in the given bin
8. For each combination of hits, compute the chi2

| parameter                  | Values      | Description |
| -------------------------- | ----------- | ----------- |
| do_efficiencies            | 0 or 1      | Compute efficiency, iff useconstraint is 1 then only points within constraint are used to compute efficiency |
| dump_geometry_info         | 0 or 1      | Print alignment info |
| efficiency_bin_width_1D    | In meters   | Bin width for 1D efficiency plots (e.g. hdidhit_x_...) |
| efficiency_bin_width_2D    | In meters   | Bin width for 1D efficiency plots (e.g. hdidhit_xy_...) |
| [x/y]pfp_[min/max]         |             | only use tracks with slopes within these bounds in track (cut on xprime and yprime respectively) |
| useslopeconstraint         | 0 or 1      | only cut on above if this is 1 |
| maxhitcombos               |             | maximum number of allowed 2D hits for a given layer (effectively maximum of N_ucluster * N_vcluster) |
| maxhitcombos_inner         |             | maximum number of allowed 2D hits in inner layers, takes precedence over maxhitcombos |
| maxhitcombos_total         |             | maximum number of allowed combinations to try and track $\prod_{layers}hits$ |
| tryfasttrack               |             | do the "fast tracking", discussed above |
| gridbinwidth[x/y]          |             | detector is 'binned' with this bin width, effectively area to search |
| gridedgetolerance[x/y]     |             | include adjacent bins if they're close |
| sigmahitpos                |             | estimate of the position resolution for chi2 calculation during track fitting |
| trackchi2cut               |             | maximum chi2/ndf computed for given track |
| minhitsontrack             | >= 3        | min number of hits to form a track |

For thinking about xprime and y prime, atan(delta [x, y] / delta z) is the corresponding angle. E.g, xp of 0.176 corresponds to 10 degrees over 1m in z.

## Other/ not relevant

These are either not relevant for MOLLER or are not current being used

| parameter              | Values   | Description |
| ---------------------- | -------- | ----------- |
| useconstraint          |          | constrain track algorithm based on other detectors (THA spectrometers) |
| constraintwidth_phi    |          | If dy/dz > this cut, we keep track but don't use it for efficiency calculation. |
| constraintwidth_theta  |          | If dx/dz > this cut, we keep track but don't use it for efficiency calculation |
| pedsub_online          | 0 or 1   | online pedestal subtraction (do not use for pedestal mode) |
| onlinezerosuppression  |          | seems like it's not being used in the code base presently |
| [x/y]ptar_[min/max]    |          | only use tracks with good slopes from target (i think negative values mean dont cut on this) |
| useopticsconstraint    | 0 or 1   | constrain based on optics, needs to be modified for MOLLER optics |

## Timing

I think the rest of the cuts are used only if "usestriptimingcut" is 1.

| parameter           | Values        | Description |
| ------------------- | ------------- | ----------- |
| usestriptimingcut   | 0, 1, or 2    | Controls whether we cut signals while decoding based on timing information |
| useTSchi2cut        | 0 or 1        | Seems like there is functionality to cut out a sample if the chi2 is greater than fStripTSchi2Cut, which is fixed at 10. This is the chi2 of a vector of time samples with respect to the "Good Strip" averages. The chi2 calculation is defined as such $\chi^2 = \sum_{\textrm{Samples}} \left( \textrm{ADC}_{i} - \textrm{ADC}_{\textrm{ max}} * \max(0, \frac{t_i - t_0}{\tau_{\textrm{strip}}}\exp(1-\frac{t_i - t_0}{\tau_{\textrm{strip}}}) \right) ^2/\sigma^2$, where it seems like $\tau$ is the "deconvolution_tau" and defaults to 56 ns. |
| maxstrip_t0         | in ns         | maximum value allowed for the t0 of the given event |
| maxstrip_tsigma     | in ns         | for purpose of "hit quality chi2" calculation |
| HitTimeMean         | in ns         | it seems like $t_0$ (defined previously) is defined with this value |
| HitTimeSigma        | in ns         | delta_t cut defined above is the max of 3.5*hittimesigma or deltat_cut |
| maxstrip_tcut       |               | We allow the mean time of the sample to be this many sigmas from the central T |
| addstrip_tcut       |               | I think this has to do with separating adjacent clusters |

## not yet investigated/understood

| parameter         | Values | Description |
| ----------------- | ------ | ----------- |
| addstrip_ccor_cut |        |             |

## resources

It seems that broadly MOLLERGEMModule.cxx controls decoding up to 2D hit reconstruction, and everything after that is handled byMOLLERGEMSpectrometerTracker.cxx and MOLLERGEMTackerBase.cxx. The latter containing the actual tracking code, and the former containing
