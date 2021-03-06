.. _host_guest_implicit:

Absolute Binding of Host cucurbit[7]uril with Guest molecule B2 in Implicit Solvent
===================================================================================

This example illustrates the computation of the binding free energy of B2 guest to the host cucurbit[7]uril in an
NVT non-periodic system of implicit water. From description, we will craft the settings, molecules, and simulations in YANK.
This example will be less detailed than the :doc:`full detailed example of p-xylene and T4 Lysozyme <p-xylene-explicit>`

This example resides in ``{PYTHON SOURCE DIR}/share/yank/examples/binding/host-guest``. The rest of the example here
assumes you are in this directory.

This example and files are based on `this AMBER tutorial <http://ambermd.org/tutorials/advanced/tutorial21/>`_. 

Examining YAML file
-------------------

Here we will look at the YAML file and options relevant to this run. We assume you have looked at the
:doc:`detailed p-xylene in T4 Lysozyme example <p-xylene-explicit>` for a more detailed explanation of the YAML file and
the settings.

Options Heading
^^^^^^^^^^^^^^^

.. code-block:: yaml

   options:
     minimize: yes
     verbose: yes
     default_number_of_iterations: 500
     temperature: 300*kelvin
     pressure: null
     output_dir: hgoutput

We choose to ``minimize`` the simulation to reduce the chance of an unstable starting configuration. Then set ``verbose``
to see what is happening.

We will be running implicit, non-periodic NVT system. So we set the ``temperature``, then set the ``pressure`` to ``null``
to ensure we are in NVT mode.

Finally, we set a limited number of iterations with ``default_number_of_iterations`` (for this example) and choose a specific
output directory called ``hgoutput``. Note that this folder is relative to the ``yank.yaml`` file.


Molecules Heading
^^^^^^^^^^^^^^^^^

.. code-block:: yaml

    molecules:
      cucurbit:
        filepath: setup/host.tripos.mol2
        leap:
          parameters: leaprc.gaff2
        antechhamber:
          charge_method: null
      B2:
        filepath: setup/guest.tripos.mol2
        antechamber:
          charge_method: bcc

Both molecules in this example are mol2 files, however, there are some nuances.

The host file has charges pre-assigned to
it, but the gaff2 force field does not know the atom names that are in the file. We pre-assign charges to molecules as
the charge assignment process can be rather taxing for large molecules, and YANK would have to do it every start up
unless molecule preparation was already done, and ``resume_simulation`` is set to ``yes``. On this host/guest system,
the process would not take long, but in general, large molecules should be pre-assigned charge. Normally, simply naming the
file with ``filepath`` would be sufficient, but since we are missing parameters in GAFF, we still invoke ``antechamber``.
Note though, the ``charge_method`` is set to ``null`` so we don't attempt to re-assign charges.

The guest molecule, B2, is identical to earlier examples and no further options are needed.


Solvents Heading
^^^^^^^^^^^^^^^^

.. code-block:: yaml

    solvents:
      GBSA:
        nonbonded_method: NoCutoff
        implicit_solvent: OBC2

The solvent in this case is an implicit/continuum dielectric solvent. We choose the options ``NoCutoff`` to mean that
all particle interactions are computed without any kind of cutoff, without periodic copes. Because there are
significantly fewer atoms in this system, we use ``NoCutoff`` to be as accurate as possible without taking too much
computational effort.

We set ``implicit_solvent`` to tell YANK to actually a continuum solvent. If we had not set this, we would have gotten
a solvent representing a vacuum. The argument given to ``implicit_solvent`` is linked to OpenMM's implicit solvent names.
In this case, the ``OBC2`` name is the Onufriev-Bashford-Case GBSA model. We could change the dielectric, but YANK
defaults to the dielectric for TIP3P water.


Systems and Protocols Headings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The headings ``systems`` and ``protocols`` are straightforward. Please see the more
:doc:`detailed p-xylene in T4 Lysozyme example <p-xylene-explicit>` for more information on these sections.

Note however in the ``protocols`` heading there is not a ``lambda_restraints`` specified, the reason for that is the
type of restraint we choose as we discuss below.

Experiments Heading
^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

    experiments:
      system: host-guest
      protocol: absolute-binding
      restraint:
        type: FlatBottom

This ``experiments`` heading differs from the :doc:`p-xylene binding example <p-xylene-explicit>` due to the type of
restraint we chose, in this case, ``FlatBottom``.

We apply a ``FlatBottom`` restraint
to keep the guest from wondering too far away from the host. The ``FlatBottom`` option applies no biasing potential
until the guest drifts too far away at which point a harmonic bias is applied relative to the geometric center of the
host. It loosely follows this equation:

.. math::


  U_{\text{restraint}}(r) =  \begin{cases}
                                               0 & \,\text{if}\, r \le r_{\text{cutoff}} \\
      K \cdot \left(r-r_{\text{cutoff}} \right)^2 & \,\text{if}\, r > r_{\text{cutoff}}
      \end{cases}

The Flat Bottom restraint has another advantage on this system in particular. The symmetric host molecule is known to
have multiple binding sites that are hard to locate in normal simulation time lengths. The ``FlatBottom`` restraint allows
the guest to explore more of the space around the host relative to the other restraint type ``Harmonic``.

The ``Harmonic``
restraint type would keep the ligand close to the centroid of the host, which may only favor one of the bindning sites.
It should be noted though, that if you want to keep the close to a the centroid, the ``Harmonic``
restraint may be a better option. We do this in the :doc:`p-xylene binding example <p-xylene-explicit>` since the binding
site is close to the centroid of the receptor and the ``Harmonic`` restraint is not so strong as to overcome
natural kinetic barriers.

We also note that we did not specify ``lambda_restraints`` because we want every state to take the same and default
value of ``1.0``. This differs from the :doc:`p-xylene binding example <p-xylene-explicit>` because we **want** the ligand
to explore the simulation box, without drifting too far away.



Running and Analyzing the Simulation
------------------------------------

The execution and analysis of the simulation are handled the same as
:doc:`in the T4 Lysozyme binding example <p-xylene-explicit>`. Please see the documentation on that page for more information.