The following are user submitted tips that may be helpful to you.

## How To Contribute

For now, please simply add in tips; as more are added, they will be organized appropriately.

## General Tips 

#### How can I cleanly shutdown a simulation (without corrupting the h5 file)?
It is generally safe to shutdown a WESTPA simulation by simply canceling the job through your queue management. However, to ensure data integrity in the h5 file, you should wait until the WESTPA log indicates that an iteration has begun or is occurring; canceling a job too quickly after submission can result in the absolute corruption of the h5 file and should be avoided.

## Dynamics Packages

WESTPA was designed to work cleanly with any dynamics package available (using the executable propagator); however, many of the tips and tricks available on the web or the user manual for these packages make the (reasonable) assumption that you will be running a set of brute force trajectories. As such, some of their guidelines for handling periodic boundary conditions may not be applicable.

### GROMACS

#### Periodic Boundary Conditions
While many of the built in tools now handle periodic boundary conditions cleanly (such as g_dist) with relatively little user interaction, others, such as g_rms, do not. If your simulation analysis protocol requires you to run such a tool, you must correct for the periodic boundary conditions before running it. While there are guidelines available to help you correct for whatever conditions your system may have [here](http://www.gromacs.org/Documentation/Terminology/Periodic_Boundary_Conditions), there is an implicit assumption that you have one long running trajectory.

It will be necessary, within your executable propagator (usually runseg.sh) to run trjconv (typically, two or three times, depending on your needs: once to remove the periodic boundary conditions, then to make molecules whole, then to remove any jumps). If no extra input is supplied (the -s flag in GROMACS 4.X), GROMACS uses the first frame of your segment trajectory as a reference state to remove jumps. If your segment's parent ended the previous iteration having jumped across the box barrier, trjconv will erroneously assume this is the correct state and 'correct' any jump back across the barrier. '''This can result in unusually high RMSD values for one segment for one or more iterations,''' and can show as discontinuities on the probability distribution. It is important to note that a lack of discontinuities does not imply a lack of imaging problems.

To fix this, simply pass in the last frame of the imaged parent trajectory and use that as the reference structure for trjconv. This will ensure that trjconv is aware if your segment has crossed the barrier at time 0 and will make the appropriate corrections.

## Development

#### I'm trying to profile a parallel script using the --profile option of bin/west. I get a PicklingError. What gives?

When executing a script using --profile, the following error may crop up:

<pre>
PicklingError: Can't pickle <type 'function'>: attribute lookup __builtin__.function failed
</pre>

The cProfile module used by the --profile option modifies function definitions such that they are no longer pickleable, meaning that they cannot be passed through the work manager to other processes. If you absolutely must profile a parallel script, use the threads work manager.