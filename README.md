# DiNo
Discretised Nonlinear Heuristic Planner (DiNo) with domain-independent planning heuristics and informed search algorithms.
DiNo supports planning for linear and nonlinear continuous PDDL+ models with processes and events, as well as Timed Initial Literals and Timed Initial Fluents.

DiNo is based on the Discretise and Validate approach and it has been designed to automatically interface with the VAL plan validator.

DiNo has been developed as an extension of UPMurphi.

## Quick Start


1. **Dependencies check** Make sure you have the required libraries: on Ubuntu you can simply issue the following command: `sudo apt-get install build-essential flex bison libc6-dev-i386 gcc-multilib g++-multilib byacc libfl-dev`. Similar packages need to be installed in other Linux distributions as well as in Cygwin under Windows.

    **Note that by default Ubuntu16.04 uses flex 2.6 which causes errors when parsing the model. Revert Flex to 2.5.39 to avoid the issue.**

2. **Download DiNo** Download the latest DiNo distribution package
`git clone https://github.com/KCL-Planning/DiNo`
3. **Compile DiNo** Type the following in the DiNo directory:
`cd src` followed by `make`.
If everything is ok, the DiNo compiler (dino) and the PDDL-to-Murphi compiler (pddl2upm) are now in the "bin" directory. 
4. **Discretise your PDDL+ Model** From the main DiNo directory invoke (dino): `bin/dino <domain.pddl> <problem.pddl>` This will call several tools (including the g++ compiler) and generate the executable planner, called problem_planner(.exe), as well as other support and intermediate files. For example:
`cd ex/linear-generator` and `../../bin/dino generator.pddl prob01.pddl`
5. **Plan** Once the executable planner has been generated, you can run it to start the planning process. By default, the planner will allocate 1Gb of memory, search for a feasible plan and write it to the file problem_plan.ddl. Example:
`./generator_planner`
6. **Validate** To use the VAL validator to automatically validate the generated plans, run DiNo with `-val` option (e.g. `./generator_planner -val`). Ready-to-use VAL (version 4) has been included in DiNo code (precompiled for Ubuntu).
To use a different version of VAL, download and install it separately, then edit the Makefile in the "src/DiNo" folder and write the complete pathname of the "validate" binary as the value of the VAL_PATHNAME constant. Find out more about VAL [here](https://github.com/KCL-Planning/VAL)

For more information about how to download, install and use DiNo email [Wiktor Piotrowski](mailto:wiktor.piotrowski@kcl.ac.uk).

## Current Limitations of DiNo

DiNo requires the PDDL+ domain **to be typed** for being processed. The only metric currently supported by DiNo is **:minimize total-time**.

## Usage Examples

### Discretisation of the PDDL+ Model

Assuming you are in the ex/linear-generator directory:

* From a PDDL+ domain and problem (with default discretisation settings):  
    `../../bin/dino generator.pddl prob01.pddl`
* From a PDDL+ domain and Problem with user specific discretisation settings: timestep 0.1, mantissa digits: 5 exponent digits: 4  
    `../../bin/dino generator.pddl prob01.pddl --custom 0.1 5 4`

### Planning and Validating

* Default settings (search for feasible solution, output PDDL+ plans, 1Gb RAM):  
    `./generator_planner`
* Specific Settings, 2Gb RAM, plan duration limited to 50 time units and PDDL+ verbose mode (includes values for the state variable):  
   `./generator_planner -m2000 -tl50 -format:pddlvv`

### Using Domain Specific Heuristics in DiNo

DiNo is equipped with a domain-independent heuristic, the SRPG+. However, domain-specific heuristics can be easily implemented in DiNo as well. There are two ways of implementing custom heuristics, temporary and permanent. 

**Temporary**

   Once the model is compiled into a C++ file (i.e. `<domain_name>.cpp` in the subfolders of the `ex` directory), the user can edit the method `void world_class::set_h_val()` and insert their own heuristics there. Simply comment out the following lines 
   
   `upm_staged_rpg::getInstance().clear_all();
    double h_val = upm_staged_rpg::getInstance().compute_rpg();`
   
and replace them with your own function for calculating the heuristic, e.g.:
   `double h_val = (1 - mu_x) + 0.1;` (note that `x` needs to be a numeric function in the PDDL domain).
   
After saving the changes in the C++ model, the model needs to be recompiled. 
IMPORTANT: do not use the `--force` flag when compiling the model after adding your heuristics, it will recompile the model from scratch and rewrite your changes with the default heuristics. 

Note, that this process will have to be repeated for every problem, as the C++ model file will be generated for each different problem file. Compiling a different problem file will cause you to lose your changes in the C++ file.

**Permanent**

   To permanently add the heuristics, the `src/cpp_code.cpp` needs to be edited. This class generates the C++ code of the model. 
   Inside the `void make_print_world(...)` method, replace the following:
   
      "  upm_staged_rpg::getInstance().clear_all();\n"	
      "  double h_val = upm_staged_rpg::getInstance().compute_rpg();\n\n"
   
   with your own function (as above), remebering to surround the function with your quotation marks, e.g.:
   
      "  double h_val = (1 - mu_x) + 0.1;\n\n"
      
   After modifying the cpp_code.cpp file, DiNo needs to be recompiled. From inside the src directory: `make cleanall; make`
      
   After this change, DiNo's default heuristic will be the new user-defined heuristic, i.e. no need to modify the C++ model files for each problem. 
   



### Papers on DiNo and UPMurphi

* Wiktor Piotrowski, Maria Fox, Derek Long, Daniele Magazzeni, Fabio Mercorio *"Heuristic Planning for PDDL+ Domains"* (2016) (Proceedings of 25th International Joint Conference on Artificial Intelligence (IJCAI 2016), 9-15/6/2016)
* Giuseppe Della Penna, Daniele Magazzeni, Fabio Mercorio *"A Universal Planning System for Hybrid Domains"* (2012) ( Applied Intelligence, vol. 36 n. 4, pp. 932 - 959, Springer DOI: 10.1007/s10489-011-0306-z	)
* Giuseppe Della Penna, Benedetto Intrigila, Daniele Magazzeni, Fabio Mercorio *"A PDDL+ Benchmark Problem: The Batch Chemical Plant"* (2010) (Proceedings of 20th International Conference on Automated Planning and Scheduling (ICAPS 2010), 12-16/5/2010, pp. 222 - 225, AAAI Press)
* Giuseppe Della Penna, Benedetto Intrigila, Daniele Magazzeni, Fabio Mercorio *"UPMurphi: a Tool for Universal Planning on PDDL+ Problems"* (2009) (Proceedings of 19th International Conference on Automated Planning and Scheduling (ICAPS 2009), 19-23/9/2009, pp. 106 - 113, AAAI Press)

