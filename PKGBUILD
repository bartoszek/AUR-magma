#!/hint/bash
#shellcheck disable=SC2034
# Maintainer: pingplug <pingplug@foxmail.com>
# Contributor: cornholio <vigo.the.unholy.carpathian@gmail.com>

##### Configuration Options
# Specify GPU compute capability Fermi (2.x) or Kepler (3.x) or Maxwell (5.x)
#_GPU_TARGET=Fermi
#_GPU_TARGET=Kepler
#_GPU_TARGET=Maxwell
#_GPU_TARGET=Pascal
#_GPU_TARGET=Volta
# Can also be one of these: sm_20 sm_30 sm_50 sm_60 sm_70 etc.
#_GPU_TARGET=sm_30
##### End

pkgname=magma
pkgver=2.5.1
pkgrel=1
pkgdesc="Provides a dense linear algebra library similar to LAPACK but for heterogeneous/hybrid architectures, starting with current 'Multicore+GPU' systems (with CUDA)"
arch=('x86_64')
url="https://icl.cs.utk.edu/magma/"
license=('custom')
depends=("cuda>=6.5.0"
         'gsl'
         'python'
         'openblas-lapack')
makedepends=('gcc-fortran'
             'cmake')
options=('staticlibs')
source=("http://icl.cs.utk.edu/projectsfiles/${pkgname}/downloads/${pkgname}-${pkgver}.tar.gz")
sha256sums=('ce32c199131515336b30c92a907effe0c441ebc5c5bdb255e4b06b2508de109f')

prepare() {
  _CMAKE_FLAGS=(\
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/magma \
  )

  if [[ -n "${_GPU_TARGET}" ]] ; then 
    _CMAKE_FLAGS+=(-DGPU_TARGET=${_GPU_TARGET})
  fi

  if [[ -f "/usr/lib/ccache/bin/nvcc-ccache" ]] ; then
    _CMAKE_FLAGS+=( \
      -DCUDA_NVCC_EXECUTABLE=/usr/lib/ccache/bin/nvcc-ccache \
    )
  fi

  if _cuda_gcc=$(basename $(readlink /opt/cuda/bin/gcc)) ; then
    if [[ -L "/usr/lib/ccache/bin/$_cuda_gcc" ]] ; then
      _CMAKE_FLAGS+=( \
        -DCUDA_HOST_COMPILER=/usr/lib/ccache/bin/$_cuda_gcc \
      )
    fi
  fi
}

build() {
  cd "${srcdir}/magma-${pkgver}"
  mkdir build-shared && pushd build-shared
  cmake \
    ${_CMAKE_FLAGS[@]} \
    -DBUILD_SHARED_LIBS:BOOL=ON \
    ..
  make magma magma_sparse
  popd
  mkdir build-static && pushd build-static
  cmake \
    ${_CMAKE_FLAGS[@]} \
    -DBUILD_SHARED_LIBS:BOOL=OFF \
    ..
  make magma magma_sparse
  popd
}

package() {
  cd "${srcdir}/magma-${pkgver}/build-shared"
  # do not build test
  sed -i "s/install: preinstall/install: magma_sparse/g" Makefile
  make DESTDIR="${pkgdir}" install
  cd "${srcdir}/magma-${pkgver}/build-static"
  # do not build test
  sed -i "s/install: preinstall/install: magma_sparse/g" Makefile
  make DESTDIR="${pkgdir}" install

  mkdir -p ${pkgdir}/opt/magma/example
  cp -ru ${srcdir}/magma-${pkgver}/example/* ${pkgdir}/opt/magma/example/
  mkdir -p ${pkgdir}/opt/magma/testing
  cp -ru ${srcdir}/magma-${pkgver}/testing/* ${pkgdir}/opt/magma/testing/
  rm -rf ${pkgdir}/opt/magma/lib/pkgconfig
  mkdir -p ${pkgdir}/usr/share/licenses/magma
  cp ${srcdir}/magma-${pkgver}/COPYRIGHT ${pkgdir}/usr/share/licenses/magma/LICENSE
}

# vim:set ts=2 sw=2 et:
