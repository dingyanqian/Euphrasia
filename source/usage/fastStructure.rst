fastStructure
=================================================

=================================================
20200122 
=================================================

Reference: 'fastStructure https://www.genetics.org/content/genetics/197/2/573.full.pdf>'_
Github download: 'fastStructure <https://rajanil.github.io/fastStructure/>'_

Github-server
--------------------------------------------------

**Getting the source code**
(in my directory -- s1950737)
::

  mkdir ~/proj
  cd ~/proj
  git clone https://github.com/rajanil/fastStructure
  cd ~/proj/fastStructure
  git fetch
  git merge origin/master #update the latest version

**Building Python extensions**
::

  locate libgsl.so  ##/usr/lib64/
  locate libgslcblas.so ##/usr/lib64/
  locate gsl/gsl_sf_psi.h ##/usr/include
  cd ../../../ #to home directory
  vim bashrc
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib64
    export CFLAGS="-I/usr/include"
    export LDFLAGS="-L/usr/lib64"
  source bashrc
  cd ~/proj/fastStructure/vars
  python setup.py build_ext --inplace

