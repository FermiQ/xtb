# Overview

This file (`src/api/calculator.f90`) defines the Fortran Application Programming Interface (API) for creating and managing single point calculators within the `xtb` project. It allows users to instantiate different calculator types (GFN-FF, GFN0-xTB, GFN1-xTB, GFN2-xTB), configure them with various parameters like solvation models, external charges, accuracy, and manage their lifecycle. This module serves as a bridge between the core `xtb` functionalities and external C-compatible interfaces.

# Key Components

This module `xtb_api_calculator` provides several public functions and a type to interact with the calculator objects.

## Type `VCalculator`
   - Description: A Fortran derived type that acts as a wrapper for a calculator object. It contains a pointer (`ptr`) to an allocatable instance of `TCalculator` (the base type for all calculators). This is used to pass calculator instances through C bindings.
   - Members:
     - `ptr`: `class(TCalculator), allocatable` - Pointer to the actual calculator object.

## Function `newCalculator_api()`
   - Description: Creates a new, empty calculator object.
   - Parameters: None.
   - Returns: `vcalc` (type `c_ptr`) - A C pointer to the newly created `VCalculator` instance.

## Subroutine `delCalculator_api(vcalc)`
   - Description: Deallocates a calculator object previously created with `newCalculator_api`.
   - Parameters:
     - `vcalc` (type `c_ptr`, intent `inout`): A C pointer to the `VCalculator` instance to be deallocated.
   - Returns: None.

## Subroutine `loadGFNFF_api(venv, vmol, vcalc, charptr)`
   - Description: Loads a GFN-FF (Grimme's Force Field) calculator.
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment` (environment object).
     - `vmol` (type `c_ptr`, value): C pointer to the `VMolecule` (molecule object).
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator` to load the GFN-FF parameters into.
     - `charptr` (character `kind=c_char`, intent `in`, optional): C pointer to a character string specifying the parameter file. Defaults to `.param_gfnff.xtb`.
   - Returns: None.

## Subroutine `loadGFN0xTB_api(venv, vmol, vcalc, charptr)`
   - Description: Loads a GFN0-xTB (Extended Tight-Binding) calculator.
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vmol` (type `c_ptr`, value): C pointer to the `VMolecule`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
     - `charptr` (character `kind=c_char`, intent `in`, optional): C pointer to the parameter file name. Defaults to `param_gfn0-xtb.txt`.
   - Returns: None.

## Subroutine `loadGFN1xTB_api(venv, vmol, vcalc, charptr)`
   - Description: Loads a GFN1-xTB calculator.
   - Parameters: (Similar to `loadGFN0xTB_api`, but defaults to `param_gfn1-xtb.txt`).
   - Returns: None.

## Subroutine `loadGFN2xTB_api(venv, vmol, vcalc, charptr)`
   - Description: Loads a GFN2-xTB calculator.
   - Parameters: (Similar to `loadGFN0xTB_api`, but defaults to `param_gfn2-xtb.txt`).
   - Returns: None.

## Subroutine `setSolvent_api(venv, vcalc, charptr, state, temperature, grid, charptr2)`
   - Description: Adds a solvation model to an already loaded calculator.
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
     - `charptr` (character `kind=c_char`, intent `in`): C pointer to the solvent name string.
     - `state` (integer `c_int`, intent `in`, optional): Solution state.
     - `temperature` (real `c_double`, intent `in`, optional): Temperature for the solvation model.
     - `grid` (integer `c_int`, intent `in`, optional): Grid level for the solvation model.
     - `charptr2` (character `kind=c_char`, intent `in`, optional): C pointer to the solvent model name string (e.g., 'gbsa', 'alpb', 'cosmo').
   - Returns: None.

## Subroutine `releaseSolvent_api(venv, vcalc)`
   - Description: Removes/deactivates the solvation model from the calculator.
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
   - Returns: None.

## Subroutine `setExternalCharges_api(venv, vcalc, npc, numbers, charges, positions)`
   - Description: Adds an external point charge potential to the calculator (supported by GFN1/2-xTB).
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
     - `npc` (integer `c_int`, intent `in`): Number of point charges.
     - `numbers` (integer `c_int`, intent `in`, array): Atomic numbers for point charges (used for hardness).
     - `charges` (real `c_double`, intent `in`, array): Magnitudes of the point charges.
     - `positions` (real `c_double`, intent `in`, array `(3, *)`): Cartesian coordinates of the point charges.
   - Returns: None.

## Subroutine `releaseExternalCharges_api(venv, vcalc)`
   - Description: Removes/deactivates the external charge potential.
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
   - Returns: None.

## Subroutine `setAccuracy_api(venv, vcalc, accuracy)`
   - Description: Sets the accuracy for the calculator.
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
     - `accuracy` (real `c_double`, value, intent `in`): Desired accuracy level.
   - Returns: None.

## Subroutine `setMaxIter_api(venv, vcalc, maxiter)`
   - Description: Sets the maximum number of iterations for iterative methods (e.g., SCF in xTB methods).
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
     - `maxiter` (integer `c_int`, value, intent `in`): Maximum number of iterations.
   - Returns: None.

## Subroutine `setElectronicTemp_api(venv, vcalc, temperature)`
   - Description: Sets the electronic temperature for methods that support it (e.g., xTB methods).
   - Parameters:
     - `venv` (type `c_ptr`, value): C pointer to the `VEnvironment`.
     - `vcalc` (type `c_ptr`, value): C pointer to the `VCalculator`.
     - `temperature` (real `c_double`, value, intent `in`): Electronic temperature in Kelvin.
   - Returns: None.

# Important Variables/Constants

- `wp` (from `xtb_mctc_accuracy`): Defines the working precision for real numbers (likely double precision).
- `pFilename` (parameters in `loadGFNFF_api`, `loadGFN0xTB_api`, etc.): Default filenames for the GFN parameter files if not provided by the user.
  - `.param_gfnff.xtb`
  - `param_gfn0-xtb.txt`
  - `param_gfn1-xtb.txt`
  - `param_gfn2-xtb.txt`

# Usage Examples

```fortran
! Fortran usage would typically involve calling these C-bound procedures
! from a C or C-compatible environment.
!
! // Example (Conceptual C-like usage)
! // #include "xtb_api.h" // Assuming a C header generated from Fortran
!
! // xtb_VEnvironment *env = xtb_newEnvironment();
! // xtb_VMolecule *mol = xtb_newMolecule(env, ...); // Simplified
! // xtb_VCalculator *calc = xtb_newCalculator();
!
! // xtb_loadGFN2xTB(env, mol, calc, NULL); // Load GFN2-xTB with default params
! // xtb_setSolvent(env, calc, "water", NULL, NULL, NULL, "gbsa");
! // double energy = xtb_getEnergy(env, mol, calc); // Assuming such a function exists for results
!
! // xtb_delCalculator(calc);
! // xtb_delMolecule(mol);
! // xtb_delEnvironment(env);
```
*Note: The primary usage of this API module is through its C bindings, intended for linking `xtb` as a library in other C/C++ or Python programs.*

# Dependencies and Interactions

- **`iso_c_binding`**: Intrinsic Fortran module used to make procedures C-interoperable.
- **`xtb_mctc_accuracy`**: Provides type definitions for precision control (e.g., `wp`).
- **`xtb_mctc_io`**: Likely used for internal I/O operations, though not directly exposed in parameter lists of these API calls.
- **`xtb_mctc_systools`**: Provides system utilities like `rdpath` for resolving file paths.
- **`xtb_api_environment`**: Provides the `VEnvironment` type and related procedures (e.g., `checkGlobalEnv`). Interacts closely as calculator operations are tied to an environment.
- **`xtb_api_molecule`**: Provides the `VMolecule` type. Calculators operate on molecular structures.
- **`xtb_api_utils`**: Potentially provides utility functions used internally.
- **`xtb_gfnff_calculator`**: Module defining the GFN-FF calculator implementation (`TGFFCalculator`, `newGFFCalculator`).
- **`xtb_xtb_calculator`**: Module defining the xTB calculator implementation (`TxTBCalculator`, `newXTBCalculator`) for GFN0, GFN1, and GFN2 methods.
- **`xtb_type_pcem`**: Provides types related to Point Charge Electrostatic embedding (`pcem`).
- **`xtb_main_setup`**: Likely contains setup routines or global configurations.
- **`xtb_solv_kernel`, `xtb_solv_input`, `xtb_solv_state`**: Modules related to the implementation of solvation models (`addSolvationModel`).
- **`xtb_type_environment`**: Defines the underlying `TEnvironment` Fortran type.
- **`xtb_type_molecule`**: Defines the underlying `TMolecule` Fortran type.
- **`xtb_type_calculator`**: Defines the base `TCalculator` Fortran type and potentially other calculator-related types.

This module acts as a high-level wrapper around the core calculator setup functionalities, preparing them for external C API calls.
```
