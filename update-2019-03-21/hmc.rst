HMC
===


Quasi-Newton HMC
----------------

- Fixed reversibility in ensemble Markov chain.
- Checked the convergence of the BFGS approximation with the true Hessian.
- Regulate the low eigenvalues of the BFGS approximation.

We observe no clear advantage of this method compared to traditional HMC
with respect to the autocorrelation of the topological charges in the 2D U(1)
lattice gauge simulations.
