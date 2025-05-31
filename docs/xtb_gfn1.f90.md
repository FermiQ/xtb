# Overview

This file (`src/xtb/gfn1.f90`) defines the GFN1-xTB (Geometry, Frequency, Noncovalent, 1st generation Extended Tight-Binding) method's parameters and provides subroutines to initialize various components of the GFN1-xTB model. It contains a large set of pre-defined arrays holding atomic and shell-specific parameters for elements up to Z=86. These parameters are used in the GFN1-xTB Hamiltonian, repulsion, dispersion, Coulomb, and halogen bond calculations.

# Key Components

The module `xtb_xtb_gfn1` primarily provides:
1.  Parameter definitions for various physical properties and model terms.
2.  An interface `initGFN1` that groups several initialization subroutines.
3.  Specific subroutines to set or calculate derived parameters.

## Interface `initGFN1`
   - Description: This public interface bundles several module procedures responsible for initializing different parts of the GFN1-xTB model data (`TxTBData` type and its components like `TRepulsionData`, `TDispersionData`, etc.).
   - Procedures:
     - `initData(self)`: Initializes the main `TxTBData` structure, setting method name, DOI, level, and calling other specific `initGFN1` procedures.
     - `initRepulsion(self)`: Initializes repulsion parameters (`TRepulsionData`).
     - `initDispersion(self)`: Initializes D3 dispersion parameters (`TDispersionData`).
     - `initCoulomb(self, nShell)`: Initializes Coulomb interaction parameters (`TCoulombData`).
     - `initHamiltonian(self, nShell)`: Initializes Hamiltonian parameters (`THamiltonianData`), including shell information, occupations, Slater exponents, etc.
     - `initHalogen(self)`: Initializes halogen bond correction parameters (`THalogenData`).

## Subroutine `setGFN1ReferenceOcc(self, nShell)`
   - Description: Sets the reference electronic occupation numbers for each shell of each element based on predefined `referenceOcc` parameters and valence shell information.
   - Parameters:
     - `self` (`type(THamiltonianData)`, `intent(inout)`): The Hamiltonian data object to be updated.
     - `nShell` (`integer`, `intent(in)`): Array containing the number of shells for each element.
   - Returns: None.

## Subroutine `setGFN1NumberOfPrimitives(self, nShell)`
   - Description: Determines and sets the number of primitive Gaussian functions for each shell of each element based on element type (H/He vs. others) and shell angular momentum.
   - Parameters:
     - `self` (`type(THamiltonianData)`, `intent(inout)`): The Hamiltonian data object.
     - `nShell` (`integer`, `intent(in)`): Array containing the number of shells for each element.
   - Returns: None.

## Subroutine `setGFN1PairParam(pairParam)`
   - Description: Initializes pair-specific interaction parameters, particularly for d-block elements.
   - Parameters:
     - `pairParam` (`real(wp)`, `allocatable`, `intent(out)`): 2D array to be filled with pair parameters.
   - Returns: None.

## Subroutine `setGFN1kCN(kCN, nShell, angShell, selfEnergy, cnShell)`
   - Description: Calculates and sets coordination number dependent scaling factors (`kCN`) for self-energies.
   - Parameters:
     - `kCN` (`real(wp)`, `intent(out)`): Output array for `kCN` values.
     - `nShell`, `angShell`, `selfEnergy`, `cnShell`: Input parameter arrays.
   - Returns: None.

## Subroutine `setGFN1ShellHardness(shellHardness, nShell, angShell, atomicHardness, angHardness)`
   - Description: Calculates shell-resolved chemical hardness values.
   - Parameters:
     - `shellHardness` (`real(wp)`, `intent(out)`): Output array for shell hardness.
     - `nShell`, `angShell`, `atomicHardness`, `angHardness`: Input parameter arrays.
   - Returns: None.

# Important Variables/Constants

This module is rich in predefined parameter arrays for elements up to Z=86 (Radon). These are critical for the GFN1-xTB model.

- **`maxElem`**: Integer parameter, defines the maximum atomic number supported (86).
- **`gfn1Globals`**: `TxTBParameter` type, holds global GFN1 scaling factors and parameters (e.g., `kshell`, `enshell`, `ipeashift`).
- **`gfn1Disp`**: `dftd_parameter` type, holds D3 dispersion correction parameters (`s6`, `s8`, `a1`, `a2`).
- **`gfn1Kinds(118)`**: Integer array, maps atomic numbers to specific kinds for parameter lookup.
- **Repulsion Parameters**:
    - `kExp`, `kExpLight`, `rExp`: Exponents for the repulsion term.
    - `repAlpha(1:maxElem)`: Element-specific repulsion exponents.
    - `repZeff(1:maxElem)`: Element-specific effective nuclear charges for repulsion.
- **Coulomb Parameters**:
    - `chemicalHardness(1:maxElem)`: Atomic chemical hardness values.
    - `thirdOrderAtom(1:maxElem)`: Atomic contributions to third-order electrostatics.
    - `shellHardness(0:2, 1:maxElem)`: Predefined shell-resolved hardness scaling factors.
- **Hamiltonian Parameters**:
    - `nShell(1:maxElem)`: Number of shells per element.
    - `angShell(3, 1:maxElem)`: Angular momentum (s,p,d) for each shell.
    - `principalQuantumNumber(3, 1:maxElem)`: Principal quantum number for each shell.
    - `referenceOcc(0:2, 1:maxElem)`: Reference electronic occupation for s, p, d shells.
    - `shellPoly(0:3, 1:maxElem)`: Coefficients for polynomials scaling Hamiltonian elements.
    - `selfEnergy(3, 1:maxElem)`: Atomic self-energies for shells.
    - `slaterExponent(3, 1:maxElem)`: Slater exponents for atomic orbitals.
- **Halogen Bond Parameters**:
    - `halogenRadScale`, `halogenDamping`: Scaling and damping factors for halogen bond correction.
    - `halogenBond(1:maxElem)`: Strength of the halogen bond interaction for specific elements.

# Usage Examples

This module is not typically used directly by end-users. Its primary role is to provide the GFN1-xTB parameters and initialization routines to the core `xtb` calculator when a GFN1-xTB calculation is requested.

```fortran
! Conceptual: How this module's initData might be called internally
! within the xtb_xtb_calculator module or similar.

! use xtb_xtb_data, only: TxTBData
! use xtb_xtb_gfn1, only: initGFN1 ! This brings in the initData procedure

! type(TxTBData) :: gfn1_model_data

! call initGFN1(gfn1_model_data) ! This resolves to calling initData(gfn1_model_data)

! Now gfn1_model_data is populated with all GFN1-xTB parameters.
```

# Dependencies and Interactions

- **`xtb_mctc_accuracy`**: Provides `wp` for working precision.
- **`xtb_param_atomicrad`**: Provides `atomicRad` (atomic radii).
- **`xtb_param_paulingen`**: Provides `paulingEN` (Pauling electronegativities).
- **`xtb_type_param`**: Defines `TxTBParameter` and `dftd_parameter` types.
- **`xtb_xtb_data`**: Defines core data structures for xTB methods like `TxTBData`, `TRepulsionData`, `TCoulombData`, `THamiltonianData`, `THalogenData`, and their respective `init` subroutines. The `initGFN1` procedures in this module populate these structures.
- **`xtb_disp_dftd3param`**: Provides D3 dispersion C6 coefficients (`copy_c6`, `reference_c6`).

This module is central to defining the GFN1-xTB method by storing and making available all its necessary parameters.
```
