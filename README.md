# Ballistic Missile Range Simulator – the new README.md

## Installation of the GUI on macOS using Anaconda (recommended)

Install Miniconda (or the full Anaconda distribution):

```
https://docs.conda.io/en/latest/miniconda.html
```

Create a new environment with Python version 3.10. If you are not familiar with Anaconda environments, follow this tutorial:

```
https://www.youtube.com/watch?v=YJC6ldI3hWk
```

Install the wxPython package:

```
conda install wxPython
```

This should install the “pythonw” executable. Unfortunately, in Anaconda environments, GUI apps can’t be executed with the usual “python” executable, we need to use “pythonw”. Make sure the “pythonw” executable is visible in your path:

```
where pythonw
```

Should give you a proper path and not return an error. Now you should be able to run the app with:

```
pythonw gui.py
```

To debug the app using Visual Studio Code, change the Python interpreter from “python” to “pythonw”.

## Installation of the GUI on macOS using the default Python distribution

Make sure you have Python 3.11 installed on your machine. You can check the version of your Python interpreter with:
```
python3 --version
```
One of the dependencies – wxPython – requires the `cmake` compiler to be present on your machine. Try:
```
clang --version
```
If (and only if) you get an error, install the Apple Developer Command Line Tools with:
```
xcode-select –install
```

Install the following Python packages:
```
pip3 install numpy
pip3 install attrdict
```
Unfortunately, `attrdict` is not compatible with Python 3.11. We have to pefrom some manual fixes. The files below are trying to `import` from `collections` within their first 10 lines of code. Replace all `from collections import` statements with `from collections.abc import` in the following files:
```
/Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/site-packages/attrdict/mapping.py

/Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/site-packages/attrdict/mixins.py

/Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/site-packages/attrdict/merge.py

/Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/site-packages/attrdict/default.py
```
Run the command below to install wxPython. Warning: This operation is computationally heavy and takes approximately 15 minutes!
```
pip3 install wxPython
```
Congratulations, you are all done now!

## Running the simulator
Go to the project folder and run the UI with:
```
python3 gui.py
```

# The original README.md (out of date)

Simulates the flight of intercontinental ballistic missiles based on launch parameters.

Written by Josh Levinger for GlobalSecurity.org in June 2005. Original credit for the simulation goes to Dr. David Wright at MIT, who wrote a version in Basic for his paper "Depressed Trajectory SLBMS", Science and Global Security, 1992, Vol 3, p101-159.

GUI updates and packaging improvements by Karsten Wolf @karstenw

### Requires:
- Python 2.7
- WxPython (compatible with v4 'Phoenix')
- NumPy

### Installation
- `pip install wxpython`
- `python gui.py`
- If running in a virtualenv, `export PYTHONHOME=$VIRTUAL_ENV` and ensure use of System python `/usr/bin/python gui.py`

## The Simulation

The equations of motion for a missile trajectory lying in a plane are given by:

![Image](range_equations.png?raw=true)

Where

* V is velocity
* T is thrust
* m is the combined mass of the stages and reentry vehicle during boost phase and the mass of a single RV after burnout
* rho is atmospheric density
* A is the cross sectional area of the booster
* h is the altitude
* g is the gravitational accleration at h
* psi is the range angle
* gamma is the range between the velocity and the local horizontal
* eta is the angle between the thrust direction and the missile axis

For gravity turn trajectories, eta is non-zero for a short period during boost and again if the orbit is depressed. For the minimum energy trajectory, eta is always zero, gamma is 90 degrees during boost, and set at each stage burnout to the optimum angle for that `gamma_burnout = 1/2 * tan-1(sin(phi)/cos(phi) - 1 - h/R_earth)`

Thrust is calculated as increasing with altitude according to normalized data from the Saturn V. For the first stage h_norm is `h / 160934 meters` (100 miles), the percent increase = `-.4339*(h_norm)3+.6233*(h_norm)2-.01*(h_norm)+1.004`. For subsequent stages, an increase of 19% over ideal is assumed.

Drag is calculated during burn as `C_drag*area*rho*V2/2` where `rho` decreases with altitude according to the barometric formula for heights less than 19,200 meters and according to the NASA’s 1976 Standard Atmospheric model for heights between 19,200 meters and 47,000 meters. `C_drag` is as calculated by Dr. David Wright for the Scud-A. Drag is neglected during re-entry, due to insufficient data on the typical RV.

## The Solver

The Advanced panel contains the ability to solve for the fuel fraction of the mis- sile, given that an approximate range is known. Using the secant form of Newton’s method (where the definition of the derivative is replaced for the df/dx), the program attempts to find the correct fuel mass and dry mass that attains the range while still summing to a known stage mass. The stage mass value does not include the payload, which is added by the simulation before beginning.

Note that the solver will occasionally fail to converge on a reasonable value. This is due to the fact that Newton’s method requires two known starting points, and often the user only knows one. The program assumes that 99% of the given value is still a valid number, but this may not be the case. If it is not, enter a larger value in the fuel fraction field before clicking Solve.

## Presets

You can edit the list of preset missile parameters, in preset.txt. The data are stored in a relatively simple format, as a dictionary of Python dictionaries. Note that fuelmass, drymass, Isp0, burntime and thrust are lists whose first (zeroth) entry is 0. This is because lists in python are zero-based, and it makes sense to track stage data by index than remembering this. The quotes around each key are also important, don't forget them if you add another preset. This file is read and eval'ed by Python, so be careful.