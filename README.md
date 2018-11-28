# de2-115-setup
tracking files used to setup my DE2-115 FPGA board
# setup
```bash
sudo apt-get install iverilog
sudo apt-get install gtkwave
```
# module for tests
```vim
module mini_counter(
input clk,
input rst, //active low as button is normally high
input en_b,
output [8:0] cnt,
output test_out,
output test_out_1
);

//reg [31:0] cntr;

reg [31:0] prescaler;
reg [31:0] cnt_internal;

always @(posedge clk)
begin
if (rst)
  prescaler =0;
else
  prescaler = prescaler +1;
end

assign test_out   = prescaler[24];
assign test_out_1 = prescaler[23];

always @(posedge clk or posedge rst)
begin
if (rst)
begin
  cnt_internal = 32'h00000000;
end
  else if (!en_b)
begin
  cnt_internal = cnt_internal + 1;
end
  end

assign cnt = cnt_internal[31:24];


endmodule
```

# Reference

https://github.com/albertxie/iverilog-tutorial


```vim

#!/usr/bin/perl
#-  ---------------------------------------------------------------------------
#-                    Copyright Message
#-  ---------------------------------------------------------------------------
#-
#-  NXP Semiconductors confidential and proprietary.
#-  COPYRIGHT   2010 by NXP Semiconductors N.V.
#-
#-  All rights are reserved. Reproduction in whole or in part is
#-  prohibited without the written consent of the copyright owner.
#-
#-  ---------------------------------------------------------------------------
#-                    Design Information
#-  ---------------------------------------------------------------------------
#-
#-  File         : $HeadURL: svn://apc5008/global/sim_scripts/trunk/sim $
#-
#-  Author       : $Author: nxa09824 $
#-
#-  Description  : Script to run ncsim simulations
#-
#-  ---------------------------------------------------------------------------
#-  $Revision: 1.21 $
#-  $Date: Wed Nov 14 04:19:14 2018 $
#-  ---------------------------------------------------------------------------

#------------------------------------------------------------------------------
# begin documentation (referenced by pod2usage)
#------------------------------------------------------------------------------

=head1 NAME

sim - run ncsim simulations

=head1 SYNOPSIS

B<sim> [-h] [-debug] [-verbose] [-image image_cond]
       [-compile | -no_compile | -elab | -cfile hdl_file | -ccode | -batch | -regression | -clean | -status]
       [-rtl] [-gate] [-gate_bc] [-gate_typ] [-gate_wc] [-gate_cond condition] [-nopg] [-pg] [-fpga]
       [-coverage module_name] [-noecho] [-m64] [-linedebug] [-parallel]
       [-bsubopts bsub_options] [-makeopts make_options] [-input sim_tcl_file] [-log logfile]
       [test_name ... | -test_list test_list_file] [-wcheck] [-ams] [-nonotifier] [-buildopt buildopt]
       [-publish_html] [-verbosity verbosity_level] [-poll_stat] [-poll_stat_time] [-remove_tmp]

=head1 DESCRIPTION

The sim script compiles and elaborates the HDL code, compiles
any test code, and invokes the ncsim simulator for the
specified chip-level tests.

This script parses the options on the command line, and invokes the makefile
$PROJ_DB/global/make/makefile with the appropriate options.

Make options may be specified on a per-test basis using makeopts files.
If a makeopts file exists in the code directory of a test, the file is
read in and the make options specified in the file are added to the make
command line for this test.  Following is an example of makeopts file
contents:

  # Example makeopts file
  # comments begin with '#'
  # make options may be specified on multiple lines
  # blank lines are ignored
  TESTBENCH=ip_tb

  BUILD=special

Make options specified using the sim -makeopts option are added to the
command line after the options in makeopts.  This allows overriding
variables specified in the makeopts file with -makeopts.

=head1 OPTIONS

Options may be specified with unique abbreviations.
For example, the -compile option can also be specified
as -com.

=over 4

=item B<-h>

Show this help page

=item B<-debug>

Print the generated command lines without running them.

=item B<-verbose>

Enable verbose messages from make.

=item B<-compile>

Compile the HDL and exit.  Useful if you only wish to verify
your HDL code compiles.

=item B<-no_compile>

Run simulation without compiling.  Useful if you have made HDL
changes, but wish to simulate without compiling them yet.

=item B<-elab>

Compile and elaborate the default configuration and exit.
Useful if you only wish to verify your HDL both compiles and elaborates.

=item B<-cfile> hdl_file

Compile file.  Compile the specified hdl_file, using the block-level
makefile where the hdl_file resides.

=item B<-ccode>

Compile test code.  Compile the test code for the specified tests and
exit.  Useful to update the compiled test code when an interactive
simulation is in progress.

=item B<-batch>

Run interactive mode with no GUI.

=item B<-regression>

Submit all specified test patterns to the regression
queue as batch jobs.  Each test pattern is submitted
as a separate job.  The default (no -regression option) is to invoke the
interactive simulation GUI for the first specified test_name

Note that when running with the -regression option, you may specify multiple
simulation builds such as -rtl, -gate, -gate_wc, etc. on the same command line.
If this is done, subprocesses, one for each build type are spawned to kick
off the simulation jobs.

=item B<-clean>

Clean up the compiled HDL for the specified build (e.g. -rtl, -gate, -gate_bc, etc.)
and exit.

=item B<-status>

Print the status of each test and exit.

=item B<-test_list> test_list_file

Operate on tests in test_list_file.
This file should contain one test pattern per line, with fields separated by a ","
The fields order is:
test_name[, buildopt, owner, associated_ticket]

Comments begin with '#'

Blank lines are ignored.

Example
test_list_file contents:

  # Example test_list_file
  i2c_cfg0

  gpio0_cfg5
  
  na_crc_test, ams, Owner

=item B<-image> image_cond

Build image with condition image_cond in images$image_cond directory and use it for simulation

=item B<-rtl>

Run rtl-level simulation.

=item B<-fpga>

Run fpga-level simulation.

=item B<-gate>

Run gate-level simulation with unit delays, no sdf annotation.

=item B<-gate_bc>

Run gate-level simulation with best case sdf.

=item B<-gate_typ>

Run gate-level simulation with typical sdf.

=item B<-gate_wc>

Run gate-level simulation with worst case sdf.

=item B<-gate_cond> condition

Run gate-level simulation with the sdf specified by condition.
For example, -gate_cond bc_cmin causes sim to pass BUILD=gate_bc_cmin
to the simulation makefile.

=item B<-nopg>

non-PG netlist is used for gate-level simulation. It can be used together with -gate* option. 
By default, PG netlist is used for gate-level simulation.

=item B<-pg>

PG netlist is used for gate-level simulation. It can be used together with -gate* option. 
By default, PG netlist is used for gate-level simulation.

=item B<-coverage> module_name

Enable elaboration and simulator command line options to collect coverage
statistics on the sub-design named module_name.  See the variable
COVERAGE in $PROJ_DB/global/make_scripts/makefile.include for more info.

=item B<-noecho>

Disable tarmac echoing to stdout and ncsim.log for RTL sims (both interactive
and regression).

=item B<-m64>

Simulate with 64bit executables(ie.ncvlog, ncvhdl, ncsim)

=item B<-wcheck>

To disable timming check by given timing file during gate-level simulation with SDF	

=item B<-ams>

Using VAMS models for most analog blocks

=item B<-nonotifier>

To use -NONOTIFIER option for ncelab for debugging purpose in back-annotated gate-level sim	 

=item B<-linedebug>

Turn on the -linedebug option for ncvhdl and ncverilog, to enable stepping through RTL
in simulation.

=item B<-parallel>

Compile test code in parallel: modifies regression process to skip compiling tests 
sequentially before submitting them to the LSF queue.

=item B<-bsubopts> bsub_options

Pass the string bsub_options to the bsub command line.

=item B<-makeopts> make_options

Pass the string make_options to the make command line.

=item B<-input> sim_tcl_file

Add a -input option to the ncsim command line, to input sim_tcl_file.
Note the path to sim_tcl_file will be relative to the global/sim_$BUILD/testname
directory where the ncsim is invoked.

=item B<-log> logfile

Write output to logfile.  By default, the output is logged
to $PROJ_DB/global/log/sim_$build.log.  Output is always echoed
to STDOUT as well.

=item B<-buildopt> buildopt

Use this switch to create and/or use a different compilation database with buildopt appended to the build name.
When used together with -test_list, will only run the tests which have the matching buildopt in the test list file
(see -test_list for details)

=item B<-publish_html>

Use this switch to create a web page that can be published with the regression results. Must be used together with -stat
In order for this switch to be used, there must exist a gen_res_page() function in publish_html_pkg.pm package under $PROJ_DB/global/bin folder,
that will create the results page in the desired format for the project.
See details about the gen_res_page parameters in its usage in report_test_status function in this script.

=item B<-verbosity>

Allows selection of testbench messaging verbosity.
See the TB messaging systemverilog file for the project for available options.

=item B<-poll_stat>

Keep poll testcases status until all tests are finished.

=item B<-poll_stat_time>

Testcases status poll interval. Default is 60s

=item B<-remove_tmp>

For testcases already finished, remove temporary files from sim_$build/<testcase> folder when doing the testcases stat. This is to save disk space during long regression runs

=back


=head1 EXAMPLES

Run interative simulation:

  sim i2c_cfg0

Run regression on two tests:

  sim -reg i2c_cfg0 gpio0_cfg5

Run both rtl and gate-level sim on two tests:

  sim -reg -rtl -gate i2c_cfg0 gpio0_cfg5

Run regression on the tests in test_list_file, without compiling any
RTL changes:

  sim -no -reg -test_list test_list_file

Ensure the project compiles, but don't launch a test:

  sim -com

Clean up all compiled HDL libraries for the RTL build:

  sim -clean -rtl

=cut

#------------------------------------------------------------------------------
# end documentation
#------------------------------------------------------------------------------

$byte_offset = -7000; # number of bytes from end of sim logfile
$end_of_file = 2;
$bsub_resource_opt = "-R rhel6";

use Pod::Usage;
use List::Util qw(first);

use lib "$ENV{PROJ_DB}/data/verification/regression";
my $publish_html_pkg_present = eval {
  require publish_html_pkg;
};

# if no args were given, print SYNOPSYS documentation section and exit
pod2usage(-verbose => 0) unless @ARGV;

# use the Getopt package to get command line arguments
use Getopt::Long;

$return_val = &GetOptions("help"            => \$help,
                          "debug"           => \$debug,
                          "verbose"         => \$verbose,
                          "compile"         => \$compile,
                          "no_compile"      => \$no_compile,
                          "elab"            => \$elab,
                          "cfile=s"         => \$cfile,
                          "ccode"           => \$ccode,
                          "image=s"         => \$image_arg,
                          "rtl"             => \$rtl,
                          "fpga"            => \$fpga,
                          "gate"            => \$gate,
                          "gate_bc"         => \$gate_bc,
                          "gate_typ"        => \$gate_typ,
                          "gate_wc"         => \$gate_wc,
                          "gate_cond=s"     => \@gate_cond,
                          "nopg"            => \$nopg,
                          "pg"              => \$pg,
                          "m64"             => \$m64,
                          "wcheck"          => \$wcheck,
                          "ams"             => \$ams,
                          "nonotifier"      => \$nonotifier,
                          "coverage=s"      => \$coverage,
                          "noecho"          => \$noecho,
                          "linedebug"       => \$linedebug,
                          "parallel"        => \$parallel,
                          "batch"           => \$batch,
                          "regression"      => \$regression,
                          "clean"           => \$clean,
                          "test_list=s"     => \$test_list_file,
                          "status"          => \$status,
                          "bsubopts=s"      => \$bsubopts,
                          "makeopts=s"      => \$makeopts_arg,
                          "input=s"         => \$input,
                          "log=s"           => \$log,
                          "cpf"             => \$cpf,
                          "no_cpf"          => \$no_cpf,
                          "buildopt=s"      => \$buildopt_arg,
                          "publish_html"    => \$publish_html,
                          "verbosity=s"     => \$verbosity_level,
                          "poll_stat"       => \$poll_stat,
                          "poll_stat_time=s"=> \$poll_stat_time,
                          "remove_tmp"      => \$remove_tmp,
                          "-definev=s"      => \$definev,
                          );

# GetOptions returns false on an error
unless ($return_val) {

  # print SYNOPSYS documentation section and exit
  pod2usage(-verbose => 0);

}

if ( $help ) {

  # print all documentation sections formatted as a man page,
  # and then exit
  pod2usage(-verbose => 2);

}

# do some command line checking
if ( ! $test_list_file && ! $compile && ! $cfile && ! $elab && ! $clean && $#ARGV == -1 ) {

  print "sim: <ERROR>: You must specify at least one test,\n";
  print "              the -test_list option, or -clean.\n";
  print "              Or, you can compile, or compile and elaborate \n";
  print "              without simulating using the -compile, -cfile, or -elab options.\n";

  # print SYNOPSYS documentation section and exit
  pod2usage(-verbose => 0);
}

# Figure out which build(s) to make

# this is done before opening the logfile, in order to figure out
# what it should be named.

$rtl_sim = 0;

if ($rtl) {
  push @builds, "rtl";
  $rtl_sim = 1;
} 

###############################################################
# FPGA
###############################################################
# FPGA based simulation. Some files are stubbed away or are fpga versions, with comparison to rtl sim.
if ($fpga) {
  push @builds, "fpga";
  $fpga_sim = 1;
} 
###############################################################

if ($gate) {
  push @builds, "gate";
} 

if ($gate_bc) {
  push @builds, "gate_bc";
} 

if ($gate_typ) {
  push @builds, "gate_typ";
}

if ($gate_wc) {
  push @builds, "gate_wc";
}

if (@gate_cond) {
  foreach $cond (@gate_cond) {
    push @builds, "gate_$cond";
  }
}

unless (@builds) {
  # default to RTL if no other builds are specified
  push @builds, "rtl";
  $rtl_sim = 1 
}

if (@builds > 1) {

  # fork a child process for each build, and wait for them to exit

  foreach $cur_build (@builds) {

    $build = $cur_build;

    $pid = fork;

    if ($pid == 0) {

      # this is a child process
      $is_child = 1;
      last;

    } else {

      # this is the parent process
      push @children, $pid;

    }

  }

  if ($pid) {

    # this is the parent process, wait for the children to finish
    print "Waiting for child processes to finish ... \n";

    foreach $child_pid (@children) {

      waitpid($child_pid, 0);

    }

    print "Done.\n";
    print "Review logfiles to see the output from the child processes.\n";

    # all children have finished, go ahead and exit
    exit 0;
  
  }

} else {

  # there is only one build, so set $build and continue without forking

  $build = shift @builds;

}

if (index($build, "gate_") != -1) {
  $cond = substr $build, 5;
} else {
  $cond = "unit";
}

$makeopts .= "BUILD=$build ";
if ((!$nopg || $pg) && !$rtl_sim && !$fpga_sim) {
  $makeopts .= "PWROPT=_pwr "; 
}

if($m64) { #invoke 64bit executables
  $makeopts .= "M64BIT=-64bit ";
}

if ($ams) {
  $makeopts .= "AMS_SIM=$ams ";
}

if ($definev) {
  $makeopts .= "DEFINEV=$definev ";
}




# Control to enable/disable CPF or override project default
if($cpf) { #enable CPF simulation
  $makeopts .= "CPF_RUN=1 ";
} else {
  if($no_cpf) { #disable CPF simulation
    $makeopts .= "CPF_RUN=0 ";
  }
}

if ($image_arg) {
  $makeopts .= "BUILDIMAGE=$image_arg ";
}

if ($verbosity_level) {
  $makeopts .= "TB_VERBOSITY=$verbosity_level ";
}

# 60s default for status polling
if (not $poll_stat_time) {
  $poll_stat_time = 60;
}

# tee STDOUT and STDERR to a log file and STDOUT

# save a copy of the STDOUT and STDERR filehandles
open SAVEOUT, ">&STDOUT";
open SAVEERR, ">&STDERR";

if ($log && ! $is_child) {

  $logfile = $log;

} else {

  # check if the log directory exists
  $logdir = "$ENV{PROJ_DB}/global/log";

  if (! -e $logdir) {

    mkdir $logdir or die "ERROR: Problem creating directory $logdir : $!\n";

  }

  if ($buildopt_arg) {
    $buildoptlog = "_$buildopt_arg";
  }
  $logfile = "$logdir/sim_${build}${buildoptlog}.log";

}

unless ($is_child) {

  # tee STDOUT to the log file
  open STDOUT, "| tee $logfile" or die "ERROR: Can't tee stdout to $logfile : $!";

} else { 

  # this is a child process
  print "Writing log to $logfile ...\n";
  open STDOUT, "> $logfile" or die "ERROR: Can't redirect stdout to $logfile : $!";

}

# redirect STDERR to STDOUT
open STDERR, ">&STDOUT";

# turn on $OUTPUT_AUTOFLUSH for STDERR and STDOUT -
# flush output after every print command
select STDERR; $| = 1;
select STDOUT; $| = 1;

# prepend $bsub_resource_opt to bsubopts
$bsubopts = "$bsub_resource_opt $bsubopts ";

my %buildopt_test;
my %test_owner;
my %test_tkt;
my %test_tkt_gate;
my %test_vector;

# build the list of tests to run
if ($test_list_file) {

  $idx_test_name = 0;
  $idx_buildopt  = 1;
  $idx_vector    = 2;
  $idx_owner     = 3;
  $idx_tkt       = 4;
  $idx_tkt_gate  = 5;

  # get the @tests list from the test_list_file
  my @fields;
  
  open(TESTFILE, $test_list_file) or die "ERROR: Couldn't open $test_list_file $!\n";

  while (<TESTFILE>) {
    s/\#.*$//;                  # remove any comments
    if (/(\w+)/) {
      @fields = split /,/;         # split separated by ","
      
      for $i (0 .. $#fields) {
        $fields[$i] =~ s/^\s+|\s+$//g;    # remove leading and trailing blanks
      }

      $run_test = 0;

      # If buildopt is defined, only run the matching tests. Otherwise run all and get buildopt from the list
      if (defined $buildopt_arg) {
        if ($fields[$idx_buildopt] eq $buildopt_arg) {
          $run_test = 1;
        }
      } else {
        $run_test = 1;
      }

      if ($run_test) {
        push(@tests, $fields[$idx_test_name]);
        if ($fields[$idx_buildopt]) {
          $buildopt_test{$fields[$idx_test_name]} = $fields[$idx_buildopt]; # add associated buildopt, if present
        }
        $buildopt_list{$fields[$idx_buildopt]} = 1;          # add buildopt to the list to be compiled
       
        $test_owner{$fields[$idx_test_name]}    = $fields[$idx_owner];
        $test_tkt{$fields[$idx_test_name]}      = $fields[$idx_tkt];
        $test_tkt_gate{$fields[$idx_test_name]} = $fields[$idx_tkt_gate];
        $test_vector{$fields[$idx_test_name]}   = $fields[$idx_vector];
      }
    }
  }

  close TESTFILE;
  
  if (!%buildopt_list) {
    die "sim: no test matching selected buildopt\n";
  }

} else {

  # get the @tests list from the command line
  @tests = @ARGV;
  $buildopt_list{$buildopt_arg} = 1;

}

# if the -verbose option is given, override the value of SILENT
if ($verbose) {

  $makeopts .= "SILENT= ";

}

if ($status) {

  report_test_status();

  sim_exit();

}


if ($cfile) {

  # figure out the absolute path to the file to be compiled
  # before changing the working directory to the build directory
  $cwd = `pwd`;
  chomp $cwd;
  $cfile = "$cwd/$cfile";

  # also, figure out the workspace root path, since it may be on a separate
  # disk partition from where PROJ_DB points
  chdir $ENV{PROJ_DB} or die "Couldn't cd to directory $ENV{PROJ_DB}: $!\n";
  $proj_db_path = `pwd`;
  chomp $proj_db_path;
}

# make will be called from inside of the build directory, so specify
# the relative path to the makefile from there
$makeopts .= "-f $ENV{PROJ_DB}/global/make/makefile ";

if ($clean) {

  $target = "clean";

# run clean locally
#  $bsubopts .= "-I ";

  run_make();

  sim_exit();

}


if ($compile) {

  $target = "compile";

  $bsubopts .= "-I ";

  run_make();

  sim_exit();

}

if ($elab) {

  print "\nCompiling and elaborating the default configuration ...\n";

  $makeopts .= "TESTNAME=default_cfg CONFIG=default_cfg ";

  $target = "elab";

  $bsubopts .= "-I ";

  # make sure testname is null before calling run_make
  # (we are not compiling for a specific test)
  $testname = "";

  run_make();

  sim_exit();

}

if ($cfile) {

  # find the makefile that corresponds to the file to be compiled
  ($parentdir, $filename) = ($cfile =~ m/(.*)\/(.*)/);

  $searchdir = $parentdir;

  # start the search one directory up from the parent directory of the file -
  # strip the last subdirectory off of searchdir
  $searchdir =~ s/(.*)\/.*/\1/;

  # verify that we are searching a subdirectory of $PROJ_DB/data
  while ($searchdir =~ m:^${proj_db_path}/data/.:) {

    if (-e "$searchdir/make/makefile") {

      $block_makefile = "$searchdir/make/makefile";
      last;

    }

    # strip the last subdirectory off of searchdir
    $searchdir =~ s/(.*)\/.*/\1/;

  }

  if (! $block_makefile) {

    die "sim: unable to find a makefile for file $cfile\n";

  }

  $target = "compile_file";

  $bsubopts .= "-I ";

  $makeopts .= "BLOCK_MAKEFILE=$block_makefile FILE=$filename ";

  run_make();

  sim_exit();

}

# set the make COVERAGE variable if the -coverage option was given
if ($coverage) {
  $makeopts .= "COVERAGE=$coverage ";
}

if ($wcheck) {
  $makeopts .= "WCHECK=$wcheck ";
}

if ($nonotifier) {
  $makeopts .= "NONOTIFIER=$nonotifier ";
}


# if the -noecho option is given, disable tarmac echoing to stdout and ncsim.log
if ($noecho) {
  $makeopts .= "TARMAC_ECHO= ";
}

if ($linedebug) {
  $makeopts .= "DEBUG=-linedebug ";
}

if ($input) {
  $makeopts .= "INPUT=$input ";
}  

    
# figure out the configuration associated with each test in @tests

# make sure the config file is up to date if this is a gate build
if ($build =~ /^gate/) {

  # call make to update the config file - don't bsub it because this
  # script is just about to read it below
  system("make $makeopts $ENV{PROJ_DB}/data/verification/testcfg/${build}.config");
  
}

# create a hash of all special configurations found in $configfile
$configfile = "$ENV{PROJ_DB}/data/verification/testcfg/${build}.config";

open CONFIGFILE, "<$configfile" || die "sim: Couldn't open $configfile: $!\n";

while (<CONFIGFILE>) {

  # match config lines of the format:
  # c : testname testval...
  # lines without at least one testval are not matched, and treated
  # the same as the default_cfg
  if (/^\s*c\s*:\s*(\S+)\s+\S+/) {
    $configs{$1} = 1;
  }

}

close CONFIGFILE;

# create the testconfigs and codedirs hashes,
# read in any makeopts files

foreach $testname (@tests) {

  $verification_codedir = "$ENV{PROJ_DB}/data/verification/testcode/$testname";
  $testsuite_codedir    = "$ENV{PROJ_DB}/global/testsuite/$testname";

  # figure out the configuration for this test
  if ($configs{$testname}) {

    # there is a special configuration for this test
    $testconfigs{$testname} = $testname;

  } elsif (-d $testsuite_codedir) {

    # this is a new testsuite test - it will be elaborated to its
    # own configuration
    $testconfigs{$testname} = $testname;

  } elsif (-d $verification_codedir) {

    # no special config, but a code directory exists -
    # use the default configuration
    $testconfigs{$testname} = "default_cfg";

  } else {

    if (not $regression) {
    die "
ERROR: No test-specific configuration was found for test $testname
 in $configfile, and the code directories:
  $verification_codedir or
  $testsuite_codedir
 do not exist.  Not a valid test.\n";
    }
  }

  # if a code directory exists for this test, save it to the codedirs hash
  if (-d $testsuite_codedir) {

    $codedirs{$testname} = $testsuite_codedir;

  } elsif (-d $verification_codedir) {

    $codedirs{$testname} = $verification_codedir;

  }

  # if a makeopts file exists for this test, read it in
  if ($codedirs{$testname} && -e "$codedirs{$testname}/makeopts") {

    read_makeopts("$codedirs{$testname}/makeopts");

  }

  # In the case we are not using a test list
  if ($buildopt_arg) {

    $buildopt_test{$testname} = $buildopt_arg;

  }

}

if ($ccode) {

  $target = "sim_setup";

  $bsubopts .= "-I ";

  # get the first test name from the @tests list
  $testname = shift @tests;

  $makeopts .= "TESTNAME=$testname ";

  if ($codedirs{$testname}) {

    # pass down the code directory path, if it exists
    $makeopts .= "CODEDIR=$codedirs{$testname} ";
    
  }

  run_make();

  sim_exit();

}

if ($clcode) {

  $target = "code_clean";

  $bsubopts .= "-I ";

  # get the first test name from the @tests list
  $testname = shift @tests;

  $makeopts .= "TESTNAME=$testname ";

  run_make();

  sim_exit();

}

if ($regression) {

  # run the tests in regression mode
  
  # Create a timestamp file for the regression. This is to be used by the HTML publishing script
  $regress_timestamp_file = "$ENV{PROJ_DB}/data/verification/regression/.regress_timestamp";
  if (! -e $regress_timestamp_file) {
    open(TIMESTAMP, '>', $regress_timestamp_file) or die "ERROR: Couldn't open regression timestamp file $regress_timestamp_file $!\n";
    my $timestamp = `date +%m%d%Y_%H%M`;   # MMDDYYYY_HHMM
    chomp $timestamp;
    print TIMESTAMP $timestamp;
    close TIMESTAMP;
  }
  
  # save the default bsub and make options
  $default_bsubopts = $bsubopts;
  $default_makeopts = $makeopts;

  # make sure all the HDL code is compiled
  print "\nMake sure HDL code is up-to-date ...\n";

  $target = "compile";

  $bsubopts .= "-I ";

  # make sure testname is null before calling run_make
  # (we are not compiling for a specific test)
  $testname = "";

  # Compile all selected buildopt
  foreach $buildopt (keys %buildopt_list) {
    $makeopts = "$default_makeopts TESTNAME=default_cfg CONFIG=default_cfg ";
    run_make();
  }

  # remove the the old .rpt and ncsim.log for each test prior to
  # running regression, to ensure sim -stat reports on up-to-date
  # info
  foreach $testname (@tests) {

    $rptdir = "$ENV{PROJ_DB}/global/sim_$build/$testname/rpt";
    unlink glob("$rptdir/*.rpt"), "$rptdir/ncsim.log";

  }

  unless ($parallel) {
    # sequentially compile any test code

    $command = "";

    foreach $testname (@tests) {

      $makeopts_build = "";
      if ($buildopt_test{$testname}) {
        $makeopts_build = "BUILDOPT=_$buildopt_test{$testname}";
      }
      
      $command .= "make $default_makeopts $makeopts_arg $makeopts_files{$testname} TESTNAME=$testname CODEDIR=$codedirs{$testname} CONFIG=$testconfigs{$testname} $makeopts_build sim_setup;\\\n";

    }

    if ($command) {

      # wait 5 seconds to ensure code updates are visible over NFS before continuing
      $command .= "sleep 5";

      print "\nCompiling test code and configurations for tests that need it ...\n";

      # the flock command ensures only one regression run will attempt to compile the tests
      # in this workspace at a time;
      # since we are the build directory (e.g. global/make/rtl), the relative path to
      # the common lock file is up one directory
      # -w 3600 = wait for 1 hour to obtain the lock before timing out
      print "flock -w 3600 $ENV{PROJ_DB}/global/make/.compile_code_lock -c 'bsub -I \"$command\"'\n";

      if (! $debug) {

        system("flock -w 3600 $ENV{PROJ_DB}/global/make/.compile_code_lock -c 'bsub -I \"$command\"'") == 0 or 
          die "\nEither flock timed out, or an error occured during code compilation.\n";

      }

    }

  }

  # set up options that are common to all tests
  $target = "sim_only";

  $workspace = $ENV{WORKSPACE_NAME};
  
  foreach $testname (@tests) {

    $simdir = "$ENV{PROJ_DB}/global/sim_$build/$testname";

    # make sure $simdir exists, so the lsf.log file can be created
    if (! -e $simdir) {

      system("mkdir -p $simdir") == 0 || die "sim: problem creating $simdir : $!\n";

    }

    print "\nRunning simulation for test $testname in $simdir ...\n";

    $bsubopts = "$default_bsubopts -oo $simdir/lsf.log -J ${testname}_${workspace}_${build} ";

    $makeopts = "$default_makeopts TESTNAME=$testname CONFIG=$testconfigs{$testname} ";

    if ($codedirs{$testname}) {

      # pass down the code directory path, if it exists
      $makeopts .= "CODEDIR=$codedirs{$testname} ";

      if (-e "$codedirs{$testname}/${testname}.sv") {
	
	# set SVTEST, if ${testname}.sv exists
	$makeopts .= "SVTEST=${testname} ";

      }

    }

# make this a tee command instead
#    $redirect = "1>$simdir/stdout.log 2>&1";

    run_make();

  }

  print "\nDone submitting tests to the queue.\n";

} else {

  # the default case - run the test in interactive mode

  # get the first test name from the @tests list
  $testname = shift @tests;

  $simdir = "$ENV{PROJ_DB}/global/sim_$build/$testname";

  $workspace = $ENV{WORKSPACE_NAME};

  print "\nRunning interactive simulation for test $testname in $simdir ...\n";

  $makeopts .= "TESTNAME=$testname CONFIG=$testconfigs{$testname} INTERACTIVE=true ";

  # add batchmode option, if specified
  if ($batch) {
    $makeopts .= "GUIFLAG=-batch ";
  }

  if ($codedirs{$testname}) {

    # pass down the code directory path, if it exists
    $makeopts .= "CODEDIR=$codedirs{$testname} ";

    if (-e "$codedirs{$testname}/${testname}.sv") {
	
      # set SVTEST, if ${testname}.sv exists
      $makeopts .= "SVTEST=${testname} ";

    }

  }

  $bsubopts .= "-I -J ${testname}_${workspace}_${build} ";

  if ($no_compile) {

    $target = "sim_only";

  } else {

    $target = "sim";

  }

  run_make();

}

# normal sim script exit
sim_exit();

sub run_make {

  if ($buildopt_test{$testname}) {
    $buildopt = $buildopt_test{$testname};
  } elsif (defined $buildopt_arg) {
    $buildopt = $buildopt_arg;
  }

  # append the buildopt
  if ($buildopt) {
    $buildopt = "_$buildopt";
    $makeopts .= "BUILDOPT=$buildopt ";
  }

  # append any test-specific make options
  $makeopts .= "$makeopts_files{$testname} ";

  # append any makeopts options given on the command-line
  $makeopts .= $makeopts_arg;

  if ($clean) {
    # run the clean command locally
    $make_command = "make $makeopts $target $redirect";
  } else {
    $make_command = "bsub $bsubopts make $makeopts $target $redirect";
  }

  print "$make_command\n";

  # create the build directory
  $builddir = "$ENV{PROJ_DB}/global/make/${build}${buildopt}";

  if (! -e $builddir) {
    print "mkdir $builddir\n";
    mkdir $builddir or die "ERROR: Problem creating directory $builddir: $!\n";
  }

  # cd to the build directory
  print "cd $builddir\n";
  chdir $builddir or die "Couldn't cd to directory $builddir: $!\n";

  if (! $debug) {

    system($make_command) == 0 or die "sim: make failed, exiting...\n";

  }
  
  $buildopt = "";

}

sub report_test_status {

  # figure out the status of the specified tests

  $workspace = $ENV{WORKSPACE_NAME};
  $report_done = 0;

  while ($report_done eq 0) {

    %test_stat = ();
    $bjobs_not_responding = 0;
    %bjobs_stat = ();
    $jobs_running = 0;
    
    # check for running or pending tests
    $return_val = open(BJOBS, "bjobs -w 2>&1 |");

    if ($return_val) {

      # process output from bjobs
      while (<BJOBS>) {

        # skip first line of output
        if (/^JOBID/) {
	        next;
        }

        # check if no jobs found
        if (/No unfinished job found/) {
	        last;
        }

        # check if bjobs is not responding
        if (/daemon not responding/) {
          $bjobs_not_responding = 1;
          last;
        }

        # split up input line into respective fields
        ($jobid, $user, $stat, $queue, $from_host,
         $exec_host, $job_name, $the_rest) = split;

        if ($job_name =~ /(.*)_${workspace}_${build}$/) {

	        $test_name = $1;

	        # record the bjobs status
	        $bjobs_stat{$test_name} = $stat;
          $jobs_running = 1;

        }

      } # while (<BJOBS>)

    } else {

      $bjobs_not_responding = 1;

    }

    if ($bjobs_not_responding) {

      warn "Warning: unable to run bjobs - any unfinished or pending\n";
      warn "         tests will default to status UNKWN.\n";

    }

    TEST: foreach $test_name (@tests) {

      # first, check if bjobs has the status of this job
      if ($bjobs_stat{$test_name}) {

        # push this test onto the corresponding status list
        push @{ $test_stat{$bjobs_stat{$test_name}} }, $test_name;

        # skip to next test
        next TEST;

      }
      
      if ($remove_tmp) {
        $tmp_files = "$ENV{PROJ_DB}/global/sim_$build/$test_name/tmp_*";
        if (-e "$ENV{PROJ_DB}/global/sim_$build/$test_name/tmp_libs") {
          system("rm -Rf $tmp_files");
        }
      }

      # check the status in the .rpt file, if it exists

      ($testrpt) = glob "$ENV{PROJ_DB}/global/sim_$build/$test_name/rpt/*.rpt";

      if (-e $testrpt) {

        open(TEST_RPT, $testrpt) || die "Problem opening $testrpt $!\n";

        while (<TEST_RPT>) {

	        if (/Status\s*:\s*Pass/) {

	          push @{ $test_stat{PASS} }, $test_name;
	          next TEST;

	        }

	        if (/Status\s*:\s*Fail/) {

	          push @{ $test_stat{FAIL} }, $test_name;
	          next TEST;

	        }

        }

      }

      # check the status in the ncsim.log file, if it exists

      $ncsim_log = "$ENV{PROJ_DB}/global/sim_$build/$test_name/rpt/ncsim.log";

      $ncsim_log_exists = 0;

      if (-e $ncsim_log) {

        $ncsim_log_exists = 1;

        open(NCSIM_LOG, $ncsim_log) || die "Problem opening $ncsim_log $!\n";

        # ncsim.log can be a big file, so jump down to $byte_offset from its end
        seek NCSIM_LOG, $byte_offset, $end_of_file;

        while (<NCSIM_LOG>) {

	        # look for the script-based tb "Pass" or "Passed" messages
	        if (/status\s*:\s*pass|pattern passed/i) {

	          push @{ $test_stat{PASS} }, $test_name;
	          next TEST;

	        }

	        # look for the script-based tb "Failed" or "FAIL!" messages
	        if (/status\s*:\s*fail|pattern failed/i) {

	          push @{ $test_stat{FAIL} }, $test_name;
	          next TEST;

	        }

          # special handling for a TDL test ending
          if (/Test completed with (\d+) (simulation)? error/) {

            if ($1) {
              # number of errors is non-zero
              push @{ $test_stat{FAIL} }, $test_name;
            } else {
              # number of errors is zero
              push @{ $test_stat{PASS} }, $test_name;
            }

            next TEST;

          }

	        if (/ncsim> (exit|finish)/) {

	          push @{ $test_stat{TIMED_OUT} }, $test_name;
	          next TEST;

	        }

        }

      }


      # handle the case where there is a ncsim.log, but we don't
      # know if the job has finished or not because bjobs is not
      # responding
      if ($bjobs_not_responding && $ncsim_log_exists ) {

        push @{ $test_stat{UNKWN} }, $test_name;
        next TEST;

      }

      # check if a problem occurred during simulation setup
      # or code compilation in the lsf log
      $lsf_log = "$ENV{PROJ_DB}/global/sim_$build/$test_name/lsf.log";

      if (-e $lsf_log) {

        open(LSF_LOG, $lsf_log) || die "Problem opening $lsf_log $!\n";

        while (<LSF_LOG>) {

	        if (/make.*\[sim_setup\] Error/) {

	          push @{ $test_stat{TEST_CODE_ERROR} }, $test_name;
	          next TEST;

	        }

        }

      }

      # if this point is reached, the test crashed for some
      # unknown reason
      push @{ $test_stat{CRASHED} }, $test_name;

    } # foreach $test_name

    # print the test status report and summary

    print "\nTest Status Report\n";
    print   "==================\n\n";

    $date = localtime;

    print "Date: $date\n\n";
    print "Simulation directory: $ENV{PROJ_DB}/global/sim_$build\n\n";

    # first print the passing tests, if any

    if ($test_stat{PASS}) {

      print_test_status_list("PASS");

    }

    # now print the status of the rest of the tests

    foreach $status_type (sort keys %test_stat) {

      if ($status_type ne "PASS") {

        print_test_status_list($status_type);

      }

    }

    # print a summary of the status of all tests

    print "Test Status Summary:\n";
    print "--------------------------------------------------------------------------------\n";

    foreach $status_type (sort keys %test_stat) {

      printf "%-20s %4d\n", $status_type, $#{ $test_stat{$status_type} }+1;

    }

    print "--------------------------------------------------------------------------------\n";

    printf "%-20s %4d\n", "Total:", $#tests+1;

    if ($publish_html) {
      if ($publish_html_pkg_present) {
        publish_html_pkg::gen_res_page($build, $cond, \%test_stat, \%test_owner, \%buildopt_test, \%test_tkt, \%test_tkt_gate, \%test_vector);
      } else {
        die "\nERROR! reading publish_html_pkg.pm\n";
      }
    }
    
    if (($jobs_running == 1) and $poll_stat) {
      sleep($poll_stat_time);  # wait 60s for next iteration
    } else {
      $report_done = 1;  # quit if no pending jobs, or html publish is not enbled
    }
  }
}


sub print_test_status_list {

  my ($status_type) = @_;

  printf "The following %d tests have a status of %s:\n", $#{ $test_stat{$status_type} }+1, $status_type;
  print "--------------------------------------------------------------------------------\n";

  foreach $test_name (sort @{ $test_stat{$status_type} }) {

    printf "%-40s %-40s\n", $test_name, $buildopt_test{$test_name};

  }

  print "\n";

}

# read in the makeopts file for the current testname
# save options found in the makeopts_files hash
sub read_makeopts {

  my ($makeopts_file) = @_;
  my @makeopts_array;

  open MAKEOPTS, $makeopts_file || die "sim : Unable to open $makeopts_file : $!\n";

  while (<MAKEOPTS>) {
    chomp;                          # Remove line feed
    s/\#.*$//;                      # remove any comments
    next if /^\s*$/;                # Ignore blank lines
    #push @makeopts_array, split;    # push any options found onto @makeopts_array
    push @makeopts_array, "$_";     # push line, keeping ""
  }

  close MAKEOPTS;

  # save the options found as a string in the makeopts_files hash
  #map {print "=== Array $_\n"} @makeopts_array;    # For debugging...
  $makeopts_files{$testname} = join ' ', @makeopts_array;

}


# successful sim script exit
sub sim_exit {

  # the tee output behaves better if the filehandles are
  # closed before exiting
  close STDERR;
  close STDOUT;

  # restore STDOUT and STDERR
  open STDOUT, ">&SAVEOUT";
  open STDERR, ">&SAVEERR";

  print "\nWrote log to $logfile\n";

  exit 0;

}

```
