.. _pesa2:

.. automodule:: neorl.hybrid.pesa2


Modern PESA (PESA2)
=================================

A module for the parallel hybrid PESA2 algorithm with prioritized experience replay from reinforcement learning. Modern PESA2 combines GWO, WOA, and DE modules in NEORL.  

Original paper: Radaideh, M. I., Shirvan, K. (2021). Prioritized Experience Replay for Parallel Hybrid Evolutionary and Swarm Algorithms: Application to Nuclear Fuel, *Under Review*.


What can you use?
--------------------

-  Multi processing: ✔️
-  Discrete spaces: ❌
-  Continuous spaces: ✔️
-  Mixed Discrete/Continuous spaces: ❌

Parameters
----------

.. autoclass:: PESA2
  :members:
  :inherited-members:
  
Example
-------

.. code-block:: python

	from neorl import PESA2
	
	#Define the fitness function
	def FIT(individual):
	        """Sphere test objective function.
	                F(x) = sum_{i=1}^d xi^2
	                d=1,2,3,...
	                Range: [-100,100]
	                Minima: 0
	        """
	        y=sum(x**2 for x in individual)
	        return y
	
	#Setup the parameter space (d=5)
	nx=5
	BOUNDS={}
	for i in range(1,nx+1):
	        BOUNDS['x'+str(i)]=['float', -100, 100]
	
	pesa2=PESA2(mode='min', bounds=BOUNDS, fit=FIT, npop=60, eta_mu=1.0, nwolves=5)
	x_best, y_best, pesa2_hist=pesa2.evolute(ngen=50, replay_every=2, verbose=1)


Notes
-----
- PESA2 is symmetric, meaning population size is equal between DE, WOA, and GWO, which is helpful to ensure that all algorithms can update the memory with similar computing time. Since GWO/WOA have typically smaller population than DE, i.e. ``nwolves < npop``, ``nwhales < npop``, PESA2 adjusts number of internal generations for GWO/WOA to ensure similar fitness calculations per individual algorithm. 
- For example, if the user sets ``npop=60`` for DE, ``nwovles=6`` for GWO, and ``nwhales=10`` for WOA, then GWO and WOA are executed internally for 10 and 6 generations, respectively, to have a total of 60 evaluations per individual algorithm.    
- ``R_frac`` defines the fraction of individuals from ``npop``, ``nwovles``, and ``nwhales`` to survive to the next generation, and also the number of samples to replay from the memory. For example, if the user sets ``R_frac=0.5``, ``npop=60`` for DE, ``nwovles=6`` for GWO, and ``nwhales=10`` for WOA, then after every generation, the top 30 individuals in DE, the best 3 wolves, and the best 5 whales survive to the next generation. Then the replay memory feeds 30 individuals to DE, three new wolves to GWO, and 5 new whales to WOA.    
- For complex problems and limited memory, we recommend to set ``memory_size ~ 5000``. When the memory gets full, old samples are overwritten by new ones. Allowing a large memory for complex problems may slow down PESA2 as handling large memories is more computationally exhaustive. If ``memory_size = None``, the memory size will be set to maximum value of ``ngen*npop*3``.
- For parallel computing of PESA2, pick ``ncores`` divisible by 3 (e.g. 6, 18, 30) to ensure equal computing power across the internal algorithms.  
- If ``ncores=1``, serial calculation of PESA2 is executed.  
- Check the sections of :ref:`GWO <gwo>`, :ref:`WOA <woa>`, and :ref:`DE <de>` for notes on the internal algorithms and the auxiliary parameters of PESA2.
- Start the prioritized replay with a small fraction for ``alpha_init < 0.1`` to increase randomness earlier to improve PESA exploration. Choose a high fraction for ``alpha_end > 0.9`` to increase exploitation by the end of evolution.
- Look for an optimal balance between ``npop`` and ``ngen``, it is recommended to minimize population size to allow for more generations.
- Total number of cost evaluations for PESA2 is ``ngen*npop*3``.