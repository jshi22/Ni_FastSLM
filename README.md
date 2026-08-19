# SLM Control

Code for driving a Hamamatsu LCOS SLM with slmsuite, used for wavefront
correction of optical tweezers in the Ni lab.

## Contents

- `slm_notebook.ipynb` — Includes hardware initialisation,
  Fourier and wavefront calibration, Zernike mask generation, beam centering,
  and the aberration measurement routines.
- `SLMSUITE_SETUP.md` — Installation and required patches for slmsuite.

## Quick start

1. Follow `SLMSUITE_SETUP.md`. Initializes this slmsuite with the required patches. 
2. `conda activate slm_env`
3. Run the initialisation cell. It opens the SLM as a mirrored display, opens
   the camera, and loads the vendor phase correction.
4. Display a blaze, change k, and confirm the first-order spot moves. If it does
   not move, the display path is broken and nothing downstream will work.

