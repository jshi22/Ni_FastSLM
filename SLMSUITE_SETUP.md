# slmsuite setup

slmsuite needs several patches before it will run with a ThorCam and a Hamamatsu
SLM. These are edits to installed files under `site-packages`, so **reinstalling
or upgrading slmsuite undoes all of them.** Pin the version, record it
here, and re-apply after any environment change.

Patched against slmsuite version: `____` (check with
`python -c "import slmsuite; print(slmsuite.__version__)"`).

Paths below are as found on this machine. Confirm them against your installed
version, since the module layout moves between releases.

## 0. Environment

    conda create -n slm_env python=3.10
    conda activate slm_env
    pip install slmsuite pyglet==2.0.0 cupy

pyglet must be 2.0.0. Later versions renamed the shader call patched in step 3.

## 1. Camera

The Python SDK supports the Thorlabs scientific camera series CC215, CS126,
CS135, CS165, CS2100, CS235, CS505, and CS895. This setup uses a CS165. Install
the Scientific Camera Interfaces package from Thorlabs before continuing.

## 2. DLL path

slmsuite does not reliably locate the ThorCam DLLs. Edit `DEFAULT_DLL_PATH` in
`Thorlabs.py` under slmsuite in site-packages to point at the Python Toolkit:

    DEFAULT_DLL_PATH = (
        r"C:\Users\M Squared\Downloads\Scientific_Camera_Interfaces_Windows-2.1"
        r"\Scientific Camera Interfaces\SDK\Python Toolkit\dlls\\"
    )

The `dlls` folder under Python Toolkit is empty by default. Copy the DLLs across
from the Native Toolkit, as the Python README instructs.

A runtime alternative exists but is not recommended, because it has to be
repeated every session:

    import os, ctypes
    DLL_DIR = os.getcwd()
    os.add_dll_directory(DLL_DIR)
    ctypes.WinDLL(os.path.join(DLL_DIR, "thorlabs_tsi_camera_sdk.dll"))

## 3. pyglet shader call

In `slmsuite/hardware/pyglet.py`, drop `blit` from the shader call:

    self.shader = pyglet.graphics.get_default_shader()

## 4. Camera bit depth

In `Thorlabs.py`, fix the constructor so bit depth comes from the camera:

    super().__init__(
        resolution=(self.cam.image_width_pixels, self.cam.image_height_pixels),
        bitdepth=self.cam.bit_depth,
        pitch_um=(self.cam.sensor_pixel_width_um, self.cam.sensor_pixel_height_um),
        name=serial,
        **kwargs
    )

## 5. Frame grab attempts

The camera often needs several tries to return a frame, and the default gives up
too early. In `Thorlabs.py`:

    def _get_image_hw(self, timeout_s=.1, trigger=True, grab=True, attempts=20):

## 6. Missing import

Add to `slmsuite/hardware/slms/slm.py`:

    from PIL import Image

## 7. Hamamatsu SLM software

No LabVIEW dependencies are needed. Open the `Installer_for_64bit_SDK` folder and
run `install`.

## Verify

Run in order:

1. `import slmsuite` succeeds.
2. The camera opens and returns a frame.
3. The SLM opens as a mirrored display on the expected display number.
4. Display a blaze, change k, and confirm the first-order spot moves. A good check that proves phase is reaching the panel as expected.
