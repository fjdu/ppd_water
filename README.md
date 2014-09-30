## License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img
alt="Creative Commons License" style="border-width:0"
src="http://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This work by
<a xmlns:cc="http://creativecommons.org/ns#"
href="https://www.lsa.umich.edu/astro/people/ci.dufujun_ci.detail"
property="cc:attributionName" rel="cc:attributionURL">Fujun Du</a> is licensed
under a <a rel="license"
href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution
4.0 International License</a>.

Recently we have published a paper (see [this
link](http://adsabs.harvard.edu/abs/2014ApJ...792....2D)) that used this code.

You are welcome to contact me at fujun.du at gmail.com or fdu at umich.edu.

## Install

Go to the ```src``` directory.  Inside it there is a ```makefile```.  You may
edit it for your own needs (but you don't have to).  Then run
```bash
    make
```
and an executable file with default name ```rac``` will be generated in the
same directory.

The ```makefile``` has a few options to compile in different environments.

#### Requirements

    1. ```Gfortran``` higher than 4.6.2 or ```Intel Fortran``` higher than
       12.1.5 (they are the version I have been using for developping the
       code).
    2. The ```cfitsio``` library.


## Run the code

There are a few input data files that are needed for the code to run.

The following files are compulsary:

    1. Configuration file.
    2. Chemical network.
    3. Initial chemical composition.
    4. Dust optical properties.

The following files are optional:

    1. Density structure.
    2. Enthalpy of formation of species.
    3. Molecular transition data.
    4. Stellar spectrum.
    5. Locations to output the intermediate steps of chemcial evolution.
    6. Species to output the intermediate steps of chemcial evolution.
    7. Species to check for grid refinement.

By default all these files are in the ```inp``` directory, though they do not
have to.  Go to this directory, and edit the file ```configure.dat``` to suit your
situation.  It has about 200 entries.  Some of them are for setting up the
physics and chemistry of the model, some are for setting up the running
environment, while others are switches telling the code whether or not it
should execute some specific tasks.  Details for editing the configure file are
included below.

After you have get the configre file ready, and have all the needed files in
place, then open a terminal and go to the directory on top of ```inp```, and
type in
```
    src/rac ./inp/configure.dat
```
to start running the code.

With the template files that are already there the code should be able to run
without any modification needed.

## Contents of configure.dat

The configuration file is in the Fortran _namelist_ format, so when editing
this file you may want to set the language type for syntax highlighting of your
editor to Fortran.

At the end of the configuration file you can write down any notes you want.
Each comment must be preceded by a "!".  They will not be read by the code.

```fortran

! All comments should be preceded by a "!".
! Inline comments should be separated from the value by at least one blank
! space.
&grid_configure
  grid_config%rmin = 1D-1  ! Grid inner boundary
  grid_config%rmax = 80D0 ! Grid outer boundary
  grid_config%zmin = 0D0   ! Grid lower boundary
  grid_config%zmax = 80D0 ! Grid upper boundary
  grid_config%dr0  = 2D-2  ! Width of the first r step
  grid_config%columnwise = .true. ! Grid arranged in column manner
  grid_config%ncol = 140 ! Number of columns
  grid_config%use_data_file_input = .false. ! Whether to load structure from a data file
  grid_config%data_dir = './inp/' ! Input dir
  grid_config%data_filename = 'RADMC_density_temperature.dat'
  grid_config%analytical_to_use = 'Andrews' ! Analytical type
  grid_config%interpolation_method = 'spline'
  grid_config%max_ratio_to_be_uniform = 1.5D0 ! Determines the coarseness of the grid
  grid_config%density_scale     = 8D0 ! Roughly the density scale you are interested in
  grid_config%density_log_range = 5D0 ! Range of density scale tou are interested in
  grid_config%max_val_considered = 1d19 ! Not used
  grid_config%min_val_considered = 1d-4 ! Min density to be considered in the first few structure iterations
  grid_config%min_val_considered_use = 1d3 ! Min density actually used
  grid_config%very_small_len = 1D-6
  grid_config%smallest_cell_size = 5D-3
  grid_config%largest_cell_size  = 5D0
  grid_config%largest_cell_size_frac  = 1D-1
  grid_config%small_len_frac = 5D-3
  grid_config%refine_at_r0_in_exp = .false.
/
&chemistry_configure
  chemsol_params%dt_first_step               = 1D-8
  chemsol_params%t_max                       = 1D6
  chemsol_params%ratio_tstep                 = 1.1D0
  chemsol_params%max_runtime_allowed         = 30.0
  chemsol_params%RTOL                        = 1D-4
  chemsol_params%ATOL                        = 1D-30
  chemsol_params%mxstep_per_interval         = 6000
  chemsol_params%chem_files_dir              = './inp/'
  chemsol_params%filename_chemical_network   = 'rate06_withgrain_lowH2Bind_hiOBind.dat'
  chemsol_params%filename_initial_abundances = 'ini_abund_waterice_meMetal.dat'
  chemsol_params%filename_species_enthalpy   = 'Species_enthalpy.dat'
  chemsol_params%H2_form_use_moeq            = .false.
  chemsol_params%flag_chem_evol_save         = .false.
  chemsol_params%evol_dust_size              = .false.
/
&heating_cooling_configure
  ! When use_analytical_CII_OI is true, the two files will not be used.
  heating_cooling_config%dir_transition_rates    = './transitions/'
  heating_cooling_config%use_analytical_CII_OI   = .true.
  heating_cooling_config%filename_CII            = ''
  heating_cooling_config%filename_OI             = ''
  heating_cooling_config%IonCoolingWithLut       = .true.
  heating_cooling_config%filename_NII            = 'N+_LUT.bin'
  heating_cooling_config%filename_SiII           = 'Si+_LUT.bin'
  heating_cooling_config%filename_FeII           = 'Fe+_LUT.bin'
  heating_cooling_config%solve_method            = 2
  heating_cooling_config%use_mygasgraincooling      = .true.
  heating_cooling_config%use_chemicalheatingcooling = .true.
  heating_cooling_config%use_Xray_heating           = .true.
  heating_cooling_config%heating_Xray_en            = 0.0D0 ! Ignored
  heating_cooling_config%heating_eff_chem           = 0.1D0
  heating_cooling_config%heating_eff_H2form         = 0.3D0
  heating_cooling_config%heating_eff_phd_H2         = 1D0
  heating_cooling_config%heating_eff_phd_H2O        = 0.3D0
  heating_cooling_config%heating_eff_phd_OH         = 0.3D0
  heating_cooling_config%cooling_gg_coeff           = 1D0
/
&montecarlo_configure
  mc_conf%nph                   = 4000000     ! Divide total star luminosity into this number.
  mc_conf%nmax_cross            = 1999999999  ! Max num of cell crossing before any ab or sc
  mc_conf%nmax_encounter        = 1999999999  ! Max num of absor and scat events
  mc_conf%ph_init_symmetric     = .true.
  mc_conf%refine_UV             = 2D-1
  mc_conf%refine_LyA            = 1D-1
  mc_conf%refine_Xray           = 1D-3
  mc_conf%disallow_any_scattering  = .false.
  mc_conf%mc_dir_in             = './inp/'
  mc_conf%mc_dir_out            = 'mc/'
  mc_conf%fname_photons         = 'escaped_photons.dat'
  mc_conf%fname_water           = 'H2O.photoxs'
  mc_conf%fname_star            = 'tw_hya_spec_combined.dat'
  mc_conf%collect_photon        = .true.
  mc_conf%collect_lam_min       = 1D0
  mc_conf%collect_lam_max       = 1D8
  mc_conf%collect_nmu           = 4
  mc_conf%collect_ang_mins      = 0D0   4D0    40D0  80D0
  mc_conf%collect_ang_maxs      = 3D0   10D0   50D0  90D0
  mc_conf%nlen_lut              = 2048
  mc_conf%TdustMin              = 1D0
  mc_conf%TdustMax              = 2D3
  mc_conf%use_blackbody_star    = .false.
/
&dustmix_configure
  dustmix_info%nmixture = 2  ! Number of mixtures you want to make
  dustmix_info%lam_min  = 1D-4 ! Minimum wavelength (micron) to be considered
  dustmix_info%lam_max  = 1D4  ! Maximum ...
  !
  dustmix_info%mix(1)%id       = 1  ! Mixture 1
  dustmix_info%mix(1)%nrawdust = 3  ! Number of raw material for mixing
  dustmix_info%mix(1)%rho      = 3D0 ! Dust material density in g cm-3
  dustmix_info%mix(1)%dir          = './inp/'
  dustmix_info%mix(1)%filenames(1) = 'silicate_draine.opti'  ! Filename of raw material 1
  dustmix_info%mix(1)%filenames(2) = 'graphite_draine_pa_0.1.opti'
  dustmix_info%mix(1)%filenames(3) = 'graphite_draine_pe_0.1.opti'
  dustmix_info%mix(1)%weights(1)   = 0.8D0  ! Weight of raw material 1
  dustmix_info%mix(1)%weights(2)   = 0.04D0
  dustmix_info%mix(1)%weights(3)   = 0.16D0
  !
  dustmix_info%mix(2)%id       = 2  ! Mixture 1
  dustmix_info%mix(2)%nrawdust = 1  ! Number of raw material for mixing
  dustmix_info%mix(2)%rho      = 3D0 ! Dust material density in g cm-3
  dustmix_info%mix(2)%dir          = './inp/'
  dustmix_info%mix(2)%filenames(1) = 'silicate_draine.opti'  ! Filename of raw material 1
  dustmix_info%mix(2)%weights(1)   = 1.0D0  ! Weight of raw material 1
/
&disk_configure
  a_disk%star_mass_in_Msun         = 0.6D0
  a_disk%star_radius_in_Rsun       = 1D0
  a_disk%star_temperature          = 4000D0
  a_disk%T_Xray                    = 1D7
  a_disk%E0_Xray                   = 0.1D0
  a_disk%E1_Xray                   = 10D0
  a_disk%lumi_Xray                 = 1.6D30
  a_disk%starpos_r                 = 0D0
  a_disk%starpos_z                 = 0D0
  !
  a_disk%use_fixed_alpha_visc      = .false.
  a_disk%allow_gas_dust_en_exch    = .false.
  a_disk%base_alpha                = 0.01D0
  !
  a_disk%Tdust_iter_tandem         = .false.
  !
  a_disk%waterShieldWithRadTran    = .true.
  !
  a_disk%andrews_gas%useNumDens    = .true. ! Gas distribution
  a_disk%andrews_gas%Md            = 2D-2   ! Total mass in Msun
  a_disk%andrews_gas%rin           = 1D-1   ! Inner edge
  a_disk%andrews_gas%rout          = 80D0   ! Outer edge
  a_disk%andrews_gas%rc            = 80D0   ! Characteristic radius
  a_disk%andrews_gas%hc            = 10D0   ! Scale height at boundary
  a_disk%andrews_gas%gam           = 1.5D0  ! Power index
  a_disk%andrews_gas%psi           = 1.0D0  !
  a_disk%andrews_gas%r0_in_exp     = 3.5D0  !
  a_disk%andrews_gas%rs_in_exp     = 1D2    !
  a_disk%andrews_gas%f_in_exp      = 1D-5   !
  !
  a_disk%ndustcompo                = 3  ! Number of dust components
  !
  a_disk%dustcompo(1)%itype        = 1     ! Dust mixture type to use
  a_disk%dustcompo(1)%mrn%rmin     = 5D-3  ! Min radius
  a_disk%dustcompo(1)%mrn%rmax     = 1D3   ! Max radius
  a_disk%dustcompo(1)%mrn%n        = 3.5D0 ! Power index
  a_disk%dustcompo(1)%andrews%useNumDens = .false.  ! Use mass density insteady of number density
  a_disk%dustcompo(1)%andrews%Md         = 5D-4
  a_disk%dustcompo(1)%andrews%rin        = 1D-1
  a_disk%dustcompo(1)%andrews%rout       = 80D0
  a_disk%dustcompo(1)%andrews%rc         = 80D0
  a_disk%dustcompo(1)%andrews%hc         = 5D0
  a_disk%dustcompo(1)%andrews%gam        = 1.5D0
  a_disk%dustcompo(1)%andrews%psi        = 1.0D0
  a_disk%dustcompo(1)%andrews%r0_in_exp  = 3.5D0
  a_disk%dustcompo(1)%andrews%rs_in_exp  = 0.5D0
  a_disk%dustcompo(1)%andrews%p_in_exp   = 3D0
  a_disk%dustcompo(1)%andrews%f_in_exp   = 1D0
  a_disk%dustcompo(1)%andrews%r0_out_exp = 60D0
  a_disk%dustcompo(1)%andrews%rs_out_exp = 5D0
  a_disk%dustcompo(1)%andrews%f_out_exp  = 1D0
  !
  a_disk%dustcompo(2)%itype        = 1      ! Dust mixture type to use
  a_disk%dustcompo(2)%mrn%rmin     = 5D-3   ! Min radius
  a_disk%dustcompo(2)%mrn%rmax     = 1D0    ! Max radius
  a_disk%dustcompo(2)%mrn%n        = 3.5D0  ! Power index
  a_disk%dustcompo(2)%andrews%useNumDens = .false. ! Use mass density insteady of number density
  a_disk%dustcompo(2)%andrews%Md         = 5D-6
  a_disk%dustcompo(2)%andrews%rin        = 1D-1
  a_disk%dustcompo(2)%andrews%rout       = 80D0
  a_disk%dustcompo(2)%andrews%rc         = 80D0
  a_disk%dustcompo(2)%andrews%hc         = 5D0
  a_disk%dustcompo(2)%andrews%gam        = 1.5D0
  a_disk%dustcompo(2)%andrews%psi        = 1.0D0
  a_disk%dustcompo(2)%andrews%r0_in_exp  = 3.5D0
  a_disk%dustcompo(2)%andrews%rs_in_exp  = 0.5D0
  a_disk%dustcompo(2)%andrews%p_in_exp   = 3D0
  a_disk%dustcompo(2)%andrews%f_in_exp   = 1D0
  !
  a_disk%dustcompo(3)%itype        = 2      ! Dust mixture type to use
  a_disk%dustcompo(3)%mrn%rmin     = 0.9D0  ! Min radius
  a_disk%dustcompo(3)%mrn%rmax     = 2D0    ! Max radius
  a_disk%dustcompo(3)%mrn%n        = 3.5D0  ! Power index
  a_disk%dustcompo(3)%andrews%useNumDens = .false. ! Use mass density insteady of number density
  a_disk%dustcompo(3)%andrews%Md         = 1D-9
  a_disk%dustcompo(3)%andrews%rin        = 1D-1
  a_disk%dustcompo(3)%andrews%rout       = 3.5D0
  a_disk%dustcompo(3)%andrews%rc         = 80D0
  a_disk%dustcompo(3)%andrews%hc         = 5D0
  a_disk%dustcompo(3)%andrews%gam        = 1.0D0
  a_disk%dustcompo(3)%andrews%psi        = 1.0D0
  a_disk%dustcompo(3)%andrews%r0_in_exp  = 0.4D0
  a_disk%dustcompo(3)%andrews%rs_in_exp  = 0.1D0
  a_disk%dustcompo(3)%andrews%p_in_exp   = 2D0
  a_disk%dustcompo(3)%andrews%f_in_exp   = 1D0
/
&raytracing_configure
  raytracing_conf%dirname_mol_data       = './transitions/'
  raytracing_conf%fname_mol_data         = 'oh2o@rovib.dat'
  raytracing_conf%line_database          = 'lamda'
  raytracing_conf%maxx                   = 140D0
  raytracing_conf%maxy                   = 140D0
  raytracing_conf%nx                     = 201
  raytracing_conf%ny                     = 201
  raytracing_conf%nfreq_window           = 1
  raytracing_conf%freq_mins              = 8.6D12
  raytracing_conf%freq_maxs              = 1D13
  raytracing_conf%nf                     = 100
  raytracing_conf%E_min                  = 0D0
  raytracing_conf%E_max                  = 5D3
  raytracing_conf%min_flux               = 1D-3  ! Jy
  raytracing_conf%save_spectrum_only     = .true.
  raytracing_conf%nlam_window            = 6
  raytracing_conf%lam_mins               = 1D-4  1D-3  0.1  1.0   10.0   100.0
  raytracing_conf%lam_maxs               = 1D-3  1D-2  1.0  10.0  100.0  1000.0
  raytracing_conf%nlam                   = 100
  raytracing_conf%abundance_factor       = 1D0
  raytracing_conf%useLTE                 = .true.
  raytracing_conf%nth                    = 1
  raytracing_conf%view_thetas            = 0D0
  raytracing_conf%dist                   = 51.0
/
&cell_configure
  cell_params_ini%omega_albedo              = 0.5D0  ! Dust albedo, only for chemistry
  cell_params_ini%UV_G0_factor_background   = 1D0    ! ISM UV
  cell_params_ini%zeta_cosmicray_H2         = 1.36D-17  ! Cosmic ray intensity
  cell_params_ini%PAH_abundance             = 1.6D-9  ! PAH abundance
  cell_params_ini%GrainMaterialDensity_CGS  = 2D0  ! Density of dust material
  cell_params_ini%MeanMolWeight             = 1.4D0
  cell_params_ini%alpha_viscosity           = 0.01D0
/
&analyse_configure
  ! Do chemical analysis for some species at some locations
  a_disk_ana_params%do_analyse                      = .true.
  a_disk_ana_params%analyse_points_inp_dir          = './inp/'
  a_disk_ana_params%file_list_analyse_points        = 'points_to_analyse.dat'
  a_disk_ana_params%file_list_analyse_species       = 'Species_to_analyse.dat'
  a_disk_ana_params%file_analyse_res_ele            = 'elemental_reservoir.dat'
  a_disk_ana_params%file_analyse_res_contri         = 'contributions.dat'
/
&iteration_configure
  a_disk_iter_params%n_iter                         = 3
  a_disk_iter_params%nlocal_iter                    = 4
  a_disk_iter_params%do_vertical_struct             = .true.
  a_disk_iter_params%do_vertical_with_Tdust         = .true.
  a_disk_iter_params%calc_Av_toStar_from_Ncol       = .false.
  a_disk_iter_params%calc_zetaXray_from_Ncol        = .false.
  a_disk_iter_params%rescale_ngas_2_rhodust         = .true.
  a_disk_iter_params%max_num_of_cells               = 10000
  !
  a_disk_iter_params%deplete_oxygen_carbon          = .false.
  a_disk_iter_params%deplete_oxygen_carbon_method   = 'vscale'
  a_disk_iter_params%gval_O                         = 1D-4
  a_disk_iter_params%vfac_O                         = 2D0
  a_disk_iter_params%gval_C                         = 1D-4
  a_disk_iter_params%vfac_C                         = 2D0
  !
  a_disk_iter_params%do_vertical_every              = 1
  a_disk_iter_params%nVertIterTdust                 = 16
  a_disk_iter_params%rtol_abun                      = 0.2D0  ! For checking convergency of a cell
  a_disk_iter_params%atol_abun                      = 1D-12
  a_disk_iter_params%n_gas_thrsh_noTEvol            = 1D15
  a_disk_iter_params%converged_cell_percentage_stop = 0.95  ! Stop if so many cells have converged.
  a_disk_iter_params%filename_list_check_refine     = 'species_check_refine.dat'  ! Do refinement based on species in this file.
  a_disk_iter_params%threshold_ratio_refine         = 10D0  ! Do refinement if the gradient (ratio) is so large
  a_disk_iter_params%nMax_refine                    = 100   ! Max times of refinments
  a_disk_iter_params%redo_montecarlo                = .true.  ! Redo Monte Carlo after each full chemcial run.
  a_disk_iter_params%flag_save_rates                = .false.  ! Whether save the calculated rates.
  a_disk_iter_params%do_continuum_transfer          = .false.
  a_disk_iter_params%do_line_transfer               = .false.
  a_disk_iter_params%backup_src                     = .true. !
  a_disk_iter_params%backup_src_cmd                 = 'find src/*.f90 src/*.f src/makefile inp/*dat inp/*opti | cpio -pdm '
  a_disk_iter_params%dump_common_dir                = '/n/Users/fdu/now/storage/data_dump_201407/'
  a_disk_iter_params%dump_sub_dir_out               = '20140930_a0/'
  a_disk_iter_params%iter_files_dir                 = '/n/Users/fdu/now/storage/201407/20140930_a0/'
/
```
