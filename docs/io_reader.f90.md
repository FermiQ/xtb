# Overview

This file (`src/io/reader.f90`) provides generic wrapper subroutines for reading molecular structures and Hessian matrices from various file formats. It acts as a unified interface to different specialized reader routines within the `xtb` codebase, abstracting the details of specific file parsers from the calling code.

# Key Components

The module `xtb_io_reader` exports two main subroutines:

## Subroutine `readMolecule(env, mol, unit, ftype)`
   - Description: Reads a molecular structure from a given file unit, determining the file type and calling the appropriate low-level parser. It populates a `TMolecule` object with the structural data.
   - Parameters:
     - `env` (`class(TEnvironment)`, `intent(inout)`): The calculation environment object, used for error reporting and other context.
     - `mol` (`class(TMolecule)`, `intent(out)`): The molecule object to be filled with data read from the file.
     - `unit` (`integer`, `intent(in)`): The Fortran file unit number from which to read the structure.
     - `ftype` (`integer`, `intent(in)`): An integer identifier specifying the expected file type (e.g., `filetype%tmol`, `filetype%xyz`). If `filetype%unknown` is provided, it defaults to `filetype%tmol`.
   - Returns: None. The `mol` object is populated directly. Errors are handled via the `env` object.

## Subroutine `readHessian(env, mol, hessian, reader, format)`
   - Description: Reads a Hessian matrix from a file using a provided `TReader` object. It supports different Hessian file formats by dispatching to specific reader functions.
   - Parameters:
     - `env` (`type(TEnvironment)`, `intent(inout)`): The calculation environment object.
     - `mol` (`type(TMolecule)`, `intent(in)`): The molecule object, used for context (e.g., number of atoms to validate Hessian dimensions).
     - `hessian` (`real(wp)`, `intent(out)`, `array(:,:)`): The output array to be filled with the Hessian matrix elements. Its shape is expected to be `(3*n, 3*n)` where `n` is the number of atoms in `mol`.
     - `reader` (`type(TReader)`, `intent(inout)`): The reader object, presumably already associated with an open file, used to perform read operations.
     - `format` (`integer`, `intent(in)`): An integer identifier for the Hessian file format (e.g., `hessType%tmol`, `hessType%orca`, `hessType%dftbplus`).
   - Returns: None. The `hessian` array is populated directly. Errors are handled via the `env` object.

# Important Variables/Constants

- `source` (character parameter within each subroutine, e.g., `"io_reader_readMolecule"`): Used for identifying the source of errors when reporting them through the `env` object.
- `wp` (from `xtb_mctc_accuracy`): Defines the working precision for real numbers (e.g., for the `hessian` matrix).
- `fileType%tmol`, `fileType%unknown` (from `xtb_mctc_filetypes`): Constants representing different molecular structure file formats.
- `hessType%tmol`, `hessType%orca`, `hessType%dftbplus` (from `xtb_mctc_filetypes`): Constants representing different Hessian file formats.

# Usage Examples

```fortran
! Conceptual example of using readMolecule
program test_reader
  use xtb_type_environment, only: TEnvironment
  use xtb_type_molecule, only: TMolecule
  use xtb_io_reader, only: readMolecule
  use xtb_mctc_filetypes, only: fileType
  implicit none

  type(TEnvironment) :: env
  type(TMolecule) :: mol
  integer :: file_unit

  ! Initialize environment (simplified)
  call env%init()

  ! Open a molecule file (e.g., coord.xyz)
  open(newunit=file_unit, file='coord.xyz', status='old', form='formatted')

  ! Read the molecule
  call readMolecule(env, mol, file_unit, fileType%xyz)

  ! Check for errors (simplified)
  if (env%is_error()) then
    write(*,*) "Error reading molecule: ", env%get_error_message()
    stop
  end if

  ! ... use the populated mol object ...
  write(*,*) "Number of atoms read: ", mol%n

  close(file_unit)
  call env%destroy()

end program test_reader
```

# Dependencies and Interactions

- **`mctc_env`**: Provides `error_type` for error handling.
- **`mctc_io`**: Provides `structure_type` and the core `read_structure` subroutine, which is the underlying workhorse for `readMolecule`.
- **`xtb_io_reader_genformat`**: Provides `readHessianDFTBPlus` for reading Hessian files in DFTB+ format.
- **`xtb_io_reader_orca`**: Provides `readHessianOrca` for reading Hessian files in ORCA format.
- **`xtb_io_reader_turbomole`**: Provides `readHessianTurbomole` for reading Hessian files in Turbomole format.
- **`xtb_mctc_accuracy`**: Provides `wp` for floating-point precision.
- **`xtb_mctc_filetypes`**: Provides `fileType` and `hessType` constants for identifying file formats.
- **`xtb_type_environment`**: Provides the `TEnvironment` type for managing calculation environment and error states.
- **`xtb_type_molecule`**: Provides the `TMolecule` type for storing molecular structure data. The `assignment(=)` interface is also used to copy `structure_type` data to `TMolecule`.
- **`xtb_type_reader`**: Provides the `TReader` type, which is used by `readHessian` to abstract file reading operations.

This module centralizes the logic for dispatching to appropriate file parsing routines based on specified formats.
```
