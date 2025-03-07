.. _ssa:

.. automodule:: neorl.evolu.ssa


Salp Swarm Algorithm (SSA)
===============================================

A module for the Salp Swarm Algorithm with parallel computing support. 

Original paper: Mirjalili, S., Gandomi, A. H., Mirjalili, S. Z., Saremi, S., Faris, H., & Mirjalili, S. M. (2017). Salp Swarm Algorithm: A bio-inspired optimizer for engineering design problems. Advances in Engineering Software, 114, 163-191.

.. image:: ../images/ssa.jpg
   :scale: 40%
   :alt: alternate text
   :align: center

What can you use?
--------------------

-  Multi processing: ✔️
-  Discrete spaces: ❌
-  Continuous spaces: ✔️
-  Mixed Discrete/Continuous spaces: ❌

Parameters
----------

.. autoclass:: SSA
  :members:
  :inherited-members:
  
Example
-------

.. code-block:: python

	from neorl import SSA
	
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
	
	nsalps=20
	#setup and evolute SSA
	ssa=SSA(mode='min', bounds=BOUNDS, fit=FIT, nsalps=nsalps, c1=None, ncores=1, seed=1)
	x_best, y_best, ssa_hist=ssa.evolute(ngen=100, verbose=1)

Notes
-----

- SSA mimics the swarming behavior of salps when navigating and foraging in oceans. SSA cares mostly about the leading salp, where its position is optimized to achieve better food source (i.e. fitness).  
- Salp leader is mostly controlled by the coefficient ``c1``, which balances SSA exploration and exploitation. The default formula for :math:`c_1 = 2e^{-\frac{4g}{ngen}}`, where :math:`g` is the current generation (goes from 1 to ``ngen``), and :math:`ngen` is the total number of generations to evolute (``ngen``). Therefore, ``c1`` is typically annealed from a large value at the beginning to increase exploration, to a very small value toward the end of evolution to prioritize exploitation. 
- The user can also provide a scalar/fixed value for ``c1`` to overwrite the default annealing formula described above. Also, the user can provide a schedule for ``c1`` generated by another formula in a ``list`` form. The size of the list MUST equal to ``ngen``. For example, for ``ngen=5``, the user can provide ``c1=[5, 0.5, 0.05, 0.005, 0.0005]``, where for every generation, the corresponding ``c1`` value is used. 
- Therefore, if ``c1=None``, the user should notice that ``ngen`` value used within the ``.evolute`` function has an impact on the ``c1`` value and hence on SSA overall performance.
- ``ncores`` argument evaluates the fitness of all salps in the swarm in parallel. Therefore, set ``ncores <= nsalps`` for most optimal resource allocation.
- Look for an optimal balance between ``nsalps`` and ``ngen``, it is recommended to minimize the number of ``nsalps`` to allow for more updates and more generations.
- Total number of cost evaluations for SSA is ``nsalps`` * ``ngen``.