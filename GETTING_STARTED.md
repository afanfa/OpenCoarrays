[This document is formatted with GitHub-Flavored Markdown.                       ]:# 
[For better viewing, including hyperlinks, read it online at                     ]:# 
[https://github.com/sourceryinstitute/opencoarrays/blob/master/GETTING_STARTED.md]:#

# [Getting Started](#getting-started) #

 *  [Compiling a CAF Program]
     * [The caf compiler wrapper]
     * [A sample basic workflow]
     * [Sample advanced workflows]
 *  [Executing a CAF Program]
     * [The cafrun program launcher]

<a name="compiling-a-caf-program">
## Compiling a CAF Program ##
</a>

<a name="the-caf-compiler-wrapper">
### The caf compiler wrapper ###
</a> 

The preferred method for compiling a CAF program is by invoking the "caf" bash script 
that the OpenCoarrays CMake scripts install in the "bin" subdirectory of the installation
path. This is an experimental script with limited but useful capabilities that will 
grow over time.  Please submit bug reports and feature requests via our [Issues] page.

The "caf" script liberates the source code and workflow from explicit dependence on the
underlying compiler and communication library in the following ways: 

1. With an OpenCoarrays-aware (OCA) CAF compiler, the "caf" script passes the unmodified
   source code to the underlying compiler with the necessary arguments for building a 
   CAF program, embedding the paths to OpenCoarrays libraries (e.g., libcaf_mpi.a) installed 
   in the "lib" subdirectory of the OpenCoarrays installation path.  The "caf" script also 
   embeds the path to the relevant module file in the "mod" subdirectory of the installation
   path (e.g., opencoarrays.mod).  This supports use association with module entities via
   ``use opencoarrays``.
2. With a non-CAF compiler (including gfortran 4.9), "caf" supports a subset of CAF by 
   replacing CAF statements with calls to procedures in the [opencoarrays module] before
   passing the source code to the compiler.  

When using GCC 4.9, we recommend using the `use` statement's "only" clause to
avoid inadvertent procedure name clashes between OpenCoarrays procedures and their 
GCC counerparts.  For example, use "use opencoarrays, only : co_reduce".

With a non-OCA and OCA CAF compilers, the extensions that "caf" imports include the collective
subroutines proposed for Fortran 2015 in the draft Technical Specification [TS 18508]
_Additional Parallel Features in Fortran_.

The latter use case provides an opportunity to mix a compiler's CAF support with that of OpenCoarrays.  
For example, a non-OCA CAF compiler, such as the Cray or Intel compilers, might support all of a 
program's coarray square-bracket syntax, while OpenCoarrays supports the same program's calls to 
collective subroutine such as `co_sum` and `co_reduce`.

<a name="a-sample-basic-workflow">
### A sample basic workflow ###
</a>

## Compiling and Running with an OpenCoarrays-aware (OCA) CAF Compiler</a> ##

Inserting the "bin" directory of the chosen installation path into the user's PATH enables the following
CAF program compilation and execution workflow:

      $ cat tally.f90 
      program main
        use iso_c_binding, only : c_int
        use iso_fortran_env, only : error_unit
        implicit none
        integer(c_int) :: tally
        tally = this_image() ! this image's contribution 
        call co_sum(tally)
        verify: block
          integer(c_int) :: image
          if (tally/=sum([(image,image=1,num_images())])) then
             write(error_unit,'(a,i5)') "Incorrect tally on image ",this_image()
             error stop
          end if
        end block verify
        ! Wait for all images to pass the test
        sync all 
        if (this_image()==1) print *,"Test passed"
      end program
      $ caf tally.f90 -o tally
      $ cafrun -np 4 ./tally
        Test passed

where "4" is the number of images to be launched at program start-up.

<a name="extending-a-non-oca-CAF-compiler"
## Extending a Non-OCA CAF Compiler</a> ##

To extend the capabilities of a non-OCA CAF compiler (e.g., the Intel or Cray compilers) 
or a non-CAF compiler (e.g., GCC 4.9), access the types and procedures of the 
[opencoarrays module] by use assocication.  For example, insert the following at line 2
in the above example:

     use opencoarrays, only : co_sum

Then compile with the "caf" compiler wrapper and launch with "cafrun" program launcher.

<a name="sample-advanced-workflow">
### A sample advanced workflows</a> ###

If the "caf" compiler wrapper cannot process the source code in question, invoke
the underlying communication library directly:

    mpif90 -fcoarray=lib -L/opt/opencoarrays/ tally.f90 \ -lcaf_mpi -o htally-I<OpenCoarrays-install-path>/mod

and also run the program with the lower-level commnication library:

    mpirun -np <number-of-images> ./tally


[Sourcery Store]: http://www.sourceryinstitute.org/store
[Virtualbox]: http://www.virtualbox.org
[Issues]: https://github.com/sourceryinstitute/opencoarrays/issues
[opencoarrays module]: ./src/extensions/opencoarrays.F90
[GCC]: http://gcc.gnu.org
[TS18508 Additional Parallel Features in Fortran]: http://isotc.iso.org/livelink/livelink?func=ll&objId=17181227&objAction=Open
[The caf compiler wrapper]: #the-caf-compiler-wrapper
[The cafrun program launcher]: #the-cafrun-program-launcher
[Compiling a CAF Program]: #compiling-a-caf-program
[A sample basic workflow]: #a-sample-basic-workflow
[Sample advanced workflows]: #sample-advanced-workflows
[Executing a CAF program]: #executing-a-caf-program