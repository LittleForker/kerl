kerl
====

Easy building and installing of Erlang/OTP instances

Kerl aims to be shell agnostic and its only dependencies, excluding what's required to actually build Erlang/OTP, are curl and git.

Unless explicitely disabled, agner is installed automatically in the sandboxes for supported Erlang/OTP versions.

All is done so that, once a specific release has been built, creating a new installation is as fast as possible.

Downloading
===========

You can download the script directly from github:

    $ curl -O https://github.com/evax/kerl/raw/master/kerl

Then ensure it is executable

    $ chmod a+x kerl

and drop it in your $PATH

How it works
============

Kerl keeps tracks of the releases it downloaded, built and installed,
allowing easy installations to new destinations (without complete rebuilding) and easy switches between Erlang/OTP installations.

Usage
=====

List the available releases (kerl ignores releases < 10):

    $ ./kerl list releases
    Getting the available releases from erlang.org...
    R10B-0 R10B-2 R10B-3 R10B-4 R10B-5 R10B-6 R10B-7 R10B-8 R10B-9 R11B-0 R11B-1
    R11B-2 R11B-3 R11B-4 R11B-5 R12B-0 R12B-1 R12B-2 R12B-3 R12B-4 R12B-5 R13A
    R13B R13B01 R13B02 R13B03 R13B04 R14A R14B R14B01 R14B02
    Run "./kerl update releases" to update this list from erlang.org

Pick your choice and build it:

    $ kerl build R14B02 r14b02
    Downloading otp_src_R14B02.tar.gz to /home/evax/.kerl/archives
    (curl progresses...)
    Verifying archive checksum...
    (curl progresses...)
    Checksum verified (229fb8f193b09ac04a57a9d7794349b7)
    Extracting source code
    Building Erlang/OTP R14B02 (r14b02), please wait...
    Erlang/OTP R14B02 has been successfully built
    Fetching and building agner...
    Agner has been successfully built

Note that named builds allow you to have different builds for the same Erlang/OTP release with different configure options:

    $ KERL_CONFIGURE_OPTIONS=--disable-hipe kerl build R14B02 r14b02_nohipe
    Verifying archive checksum...
    Checksum verified (229fb8f193b09ac04a57a9d7794349b7)
    Extracting source code
    Building Erlang/OTP R14B02 (r14b02_nohipe), please wait...
    Erlang/OTP R14B02 (r14b02_nohipe) has been successfully built
    Fetching and building agner...
    Agner has been successfully built

You can verify your build has been registered:

    $ kerl list builds
    R14B02,r14b02
    R14B02,r14b02_nohipe

Now install a build to some location (optionally you can disable agner support by adding KERL_DISABLE_AGNER=yes to your $HOME/.kerlrc file, or on the contrary define a list of additional packages to install using the KERL_AGNER_AUTOINSTALL directive in the same file):
   
    $ kerl install r14b02 /path/to/install/dir/
    Installing Erlang/OTP R14B02 (r14b02) in /path/to/install/dir...
    Installing agner in /path/to/install/dir...
    You can activate this installation running the following command:
    . /path/to/install/dir/activate
    Later on, you can leave the installation typing:
    kerl_deactivate

Here again you can check the installation's been registered:

    $ kerl list installations
    r14b02 /path/to/install/dir

And at last activate it:

    $ . /path/to/install/dir/activate

Activation will backup your $PATH, prepend it with the installation's bin/ directory and do the right thing with AGNER_* vars to constrain agner operations to the sandbox. Thus it's only valid for the current shell session, and until you activate another installation or call kerl_deactivate.

You're now ready to work with your r14b02 installation:

    $ erl -version
    Erlang (SMP,ASYNC_THREADS,HIPE) (BEAM) emulator version 5.8.3

    $ agner version
    0.4.16

You can use agner to install packages in your activated installation, they'll be directly available:

    $ agner install cowboy
    (...)
    Installed to:
    /path/to/install/dir/lib/cowboy-@master

    $ erl
    (...)
    Eshell V5.8.3  (abort with ^G)
    1> application:start(cowboy).
    ok

Note that you can also define a list of packages to be autoinstalled on agner enabled installations using the KERL_AGNER_AUTOINSTALL entry in you $HOME/.kerlrc file

When your done just type:
    $ kerl_deactivate

Anytime you can check which installation, if any, is currently active with:

    $ kerl active
    No Erlang/OTP kerl installation is currently active

You can get an overview of the current kerl state with:

    $ kerl status
    Available builds:
    R14B02,r14b02
    R14B02,r14b02_nohipe
    ----------
    Available installations:
    r14b02 /path/to/install/dir
    ----------
    Currently active installation:
    The current active installation is:
    /path/to/install/dir

You can delete builds and installations with the following commands:
    
    $ kerl delete build r14b02
    The r14b02 build has been deleted

    $ kerl delete installation /path/to/install/dir
    The installation in /path/to/install/dir has been deleted

You can update the agner version associated with a specific build (this will only affect installations made after that):

    $ kerl update agner r14b02
    Updating agner for build r14b02...
    agner has been updated successfully

Tuning
======

You can tune kerl using the .kerlrc file in your $HOME directory.

You can set the following variables:

- KERL_DOWNLOAD_DIR where to put downloaded files, defaults to $HOME/.kerl/archives
- KERL_BUILD_DIR where to hold the builds, defaults to $HOME/.kerl/builds
- KERL_CONFIGURE_OPTIONS options to pass to Erlang's ./configure script, e.g. --without-termcap
- KERL_DISABLE_AGNER if non-empty will disable agner support
- KERL_AGNER_AUTOINSTALL a list of packages to pre-install
- KERL_SASL_STARTUP use SASL system startup instead of minimal
- KERL_USE_AUTOCONF use autoconf in the builds process

