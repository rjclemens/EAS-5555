**Logging In to Derecho**
1. ```ssh -x -y clemensr@derecho.hpc.ucar.edu```
2. Enter password, Duo authenticate

**Copying WFS and WPS Files**
1. ***Login*** to Derecho
2. ```cd /glade/work/wrfhelp/```
3. ```cp -rp wrfv4.1 wpsv4.1 ~/```

**Hurricane Matthew WRF Quickstart**
1. ***Login*** to Derecho
2. Make a project directory: ```mkdir ~/EAS-5555```
3. ***Copy*** WRS and WPS Files
4. Retrieve simulation data: 
  a. ```mkdir EAS-5555/DATA```
  b. ```cd DATA```
  c. ```wget https://www2.mmm.ucar.edu/wrf/TUTORIAL_DATA/matthew_1deg.tar.gz```
  d. ```tar -xvf matthew_1deg.tar.gz```
5. Symbolic link the Vtable into WPSv4.1:
  a. ```cd ~/EAS-5555/wpsv4.1/```
  b. ```ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable```
6. Run `link_grib.csh`
  a. ```./link_grib.csh ../DATA/matthew/fnl```
  b. ```ls -alstr```
  c. You should see `GRIBFILE.AA*` in the WPS directory.
7. Rewrite `namelist.wps`
  a. Paste the following text: <details close> 
 &share
  wrf_core = 'ARW',
  max_dom = 1,
  start_date = '2016-10-06_00:00:00',
  end_date   = '2016-10-08_00:00:00',
  interval_seconds = 21600
  io_form_geogrid = 2,
 /
 &geogrid
  parent_id         =   1,
  parent_grid_ratio =   1,
  i_parent_start    =   1,
  j_parent_start    =   1,
  e_we              =  91
  e_sn              =  100
 !
 !!!!!!!!!!!!!!!!!!!!!!!!!!!! IMPORTANT NOTE !!!!!!!!!!!!!!!!!!!!!!!!!!!!
 ! The default datasets used to produce the MAXSNOALB and ALBEDO12M
 ! fields have changed in WPS v4.0. These fields are now interpolated
 ! from MODIS-based datasets.
 !
 ! To match the output given by the default namelist.wps in WPS v3.9.1,
 ! the following setting for geog_data_res may be used:
 !
 ! geog_data_res = 'maxsnowalb_ncep+albedo_ncep+default', 'maxsnowalb_ncep+albedo_ncep+default', 
 !
 !!!!!!!!!!!!!!!!!!!!!!!!!!!! IMPORTANT NOTE !!!!!!!!!!!!!!!!!!!!!!!!!!!!
 !
 geog_data_res = 'default'
 dx = 27000,
 dy = 27000,
 map_proj = 'mercator',
 ref_lat   =  28.00,
 ref_lon   = -75.00,
 truelat1  =  30.0,
 truelat2  =  60.0,
 stand_lon = -75.0,
 geog_data_path = '/glade/work/wrfhelp/WPS_GEOG/'
/
&ungrib
 out_format = 'WPS',
 prefix = 'FILE',
/
&metgrid
 fg_name = 'FILE'
 io_form_metgrid = 2, 
/

8. Run `ungrib.exe`: ```./ungrib.exe```
  a. Wait for the message `Successful completion of ungrib` to appear.

9. Run `geogrid.exe`: ```./geogrid.exe```
  a. Wait a few minutes for the message `Successful completion of geogrid` to appear.

10. Run interpolator `metgrid.exe`: ```./metgrid.exe```
  a. Wait a few minutes for the message `Successful completion of metgrid` to appear.
  b. Look for files of name `met_em.d01.2016-10-*`

11. Setup `wrf.exe` parameters:
  a. `cd ~/EAS-5555/wrfv4.1/test/em_real/`
  b. `ln -sf ~/EAS-5555/wpsv4.1/met_em.d01.2016-10* .`
  c. `vim namelist.input`
  d. Rewrite `namelist.input` to: <details close>
   &time_control
  run_days                            = 0,
  run_hours                           = 48,
  run_minutes                         = 0,
  run_seconds                         = 0,
  start_year = 2016,
  start_month = 10,
  start_day = 06,
  start_hour = 00,
  end_year = 2016,
  end_month = 10,
  end_day = 08,
  end_hour = 00,
  interval_seconds = 21600
  input_from_file = .true.,
  history_interval = 180,
  frames_per_outfile = 1,
  restart = .false.,
  restart_interval = 1440,
  io_form_history                     = 2
  io_form_restart                     = 2
  io_form_input                       = 2
  io_form_boundary                    = 2
  /
  &domains
  time_step                           = 150,
  time_step_fract_num                 = 0,
  time_step_fract_den                 = 1,
  max_dom                             = 1,
  e_we = 91,
  e_sn = 100,
  e_vert = 45,
  p_top_requested                     = 5000,
  num_metgrid_levels                  = 32,
  num_metgrid_soil_levels             = 4,
  dx                                  = 27000,
  dy                                  = 27000,
  grid_id                             = 1,     2,     3,
  parent_id                           = 0,     1,     2,
  i_parent_start                      = 1,     31,    30,
  j_parent_start                      = 1,     17,    30,
  parent_grid_ratio                   = 1,     3,     3,
  parent_time_step_ratio              = 1,     3,     3,
  feedback                            = 1,
  smooth_option                       = 0
  /
  &physics
  physics_suite                       = 'CONUS'
  mp_physics                          = -1,    -1,    -1,
  cu_physics                          = -1,    -1,     0,
  ra_lw_physics                       = -1,    -1,    -1,
  ra_sw_physics                       = -1,    -1,    -1,
  bl_pbl_physics                      = -1,    -1,    -1,
  sf_sfclay_physics                   = -1,    -1,    -1,
  sf_surface_physics                  = -1,    -1,    -1,
  radt                                = 30,    30,    30,
  bldt                                = 0,     0,     0,
  cudt                                = 5,     5,     5,
  icloud                              = 1,
  num_land_cat                        = 21,
  sf_urban_physics                    = 0,     0,     0,
  /
  &fdda
  /
  &dynamics
  hybrid_opt                          = 2, 
  w_damping                           = 0,
  diff_opt                            = 1,      1,      1,
  km_opt                              = 4,      4,      4,
  diff_6th_opt                        = 0,      0,      0,
  diff_6th_factor                     = 0.12,   0.12,   0.12,
  base_temp                           = 290.
  damp_opt                            = 3,
  zdamp                               = 5000.,  5000.,  5000.,
  dampcoef                            = 0.2,    0.2,    0.2
  khdif                               = 0,      0,      0,
  kvdif                               = 0,      0,      0,
  non_hydrostatic                     = .true., .true., .true.,
  moist_adv_opt                       = 1,      1,      1,     
  scalar_adv_opt                      = 1,      1,      1,     
  gwd_opt                             = 1,
  /
  &bdy_control
  spec_bdy_width                      = 5,
  specified                           = .true.
  /
  &grib2
  /
  &namelist_quilt
  nio_tasks_per_group = 0,
  nio_groups = 1,
  /


12. Run simulation:
  a. `./real.exe`
  b. Search error log for `SUCCESS COMPLETE REAL_EM_INIT`: `cat rsl.error.0000`
  c. `./wrf.exe`