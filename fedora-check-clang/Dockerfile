FROM registry.fedoraproject.org/fedora:42

# Set locale / timezone
RUN dnf -y install glibc-langpack-en && \
    ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
ENV TZ=Europe/London \
    LC_ALL=en_GB.UTF-8 \
    LC_COLLATE=C

# Install core packages 
RUN dnf -y --refresh \
           --setopt=skip_if_unavailable=true \
    install \
        gcc gcc-c++ gcc-gfortran make git curl \
        blas-devel lapack-devel openblas-devel \
        pcre2-devel bzip2-devel xz-devel zlib-devel libzstd-devel \
        libcurl-devel openssl-devel libxml2-devel readline-devel \
        cairo-devel pango-devel harfbuzz-devel fribidi-devel \
        fontconfig-devel freetype-devel glib2-devel libicu-devel \
        libjpeg-turbo-devel libpng-devel libtiff-devel \
        libX11-devel libXt-devel libXmu-devel libXext-devel \
        libSM-devel libICE-devel libXrender-devel \
        libtirpc-devel libxcrypt-devel ncurses-devel \
        tcl-devel tk-devel pkgconf-pkg-config \
        texlive-scheme-full ghostscript java-21-openjdk-devel \
        cmake ninja-build python3 python3-devel libstdc++-devel \
	&& dnf clean all

# Set flags and path for clang
RUN dnf -y install clang llvm lld libcxx libcxx-devel libcxxabi libcxxabi-devel libunwind
ENV CXX="clang++ -stdlib=libc++" \
    CPPFLAGS="-isystem /usr/include/c++/v1" \
    LDFLAGS="-lc++ -lc++abi"

ENV PATH="/usr/local/clang20/bin:${PATH}" 
ENV LD_LIBRARY_PATH="/usr/local/clang20/lib:/usr/local/clang20/lib64"

# Install pandoc 
RUN curl -L https://github.com/jgm/pandoc/releases/download/3.7.0.2/pandoc-3.7.0.2-linux-amd64.tar.gz \
  | tar xz -C /usr/local --strip-components=1

# Install libdeflate 1.24 
RUN curl -fsSL \
      https://github.com/ebiggers/libdeflate/releases/download/v1.24/libdeflate-1.24.tar.gz \
      | tar -xz && \
    cd libdeflate-1.24 && \
    cmake -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local && \
    cmake --build build --parallel && \
    cmake --install build && \
    cd / && rm -rf libdeflate-1.24

# Install R-devel 
COPY config.site /usr/local/share/
RUN curl -fsSL https://cran.r-project.org/src/base-prerelease/R-devel.tar.gz \
     | tar -xz && \
    cd R-devel && \
    ./configure --without-lapack --prefix=/usr/local --enable-R-shlib --with-blas && \
    make -j$(nproc) && make install && \
    cd / && rm -rf R-devel

# Set CRAN ENVVARS and parallelism
ENV _R_CHECK_INSTALL_DEPENDS_=true \
    _R_CHECK_SUGGESTS_ONLY_=true \
    _R_CHECK_NO_RECOMMENDED_=true \
    _R_CHECK_DOC_SIZES2_=true \
    _R_CHECK_DEPRECATED_DEFUNCT_=true \
    _R_CHECK_SCREEN_DEVICE_=warn \
    _R_CHECK_REPLACING_IMPORTS_=true \
    _R_CHECK_TOPLEVEL_FILES_=true \
    _R_CHECK_DOT_FIRSTLIB_=true \
    _R_CHECK_RD_LINE_WIDTHS_=true \
    _R_CHECK_S3_METHODS_NOT_REGISTERED_=true \
    _R_CHECK_OVERWRITE_REGISTERED_S3_METHODS_=true \
    _R_CHECK_CODE_USAGE_WITH_ONLY_BASE_ATTACHED_=TRUE \
    _R_CHECK_NATIVE_ROUTINE_REGISTRATION_=true \
    _R_CHECK_FF_CALLS_=registration \
    _R_CHECK_PRAGMAS_=true \
    _R_CHECK_COMPILATION_FLAGS_=true \
    _R_CHECK_R_DEPENDS_=true \
    _R_CHECK_PACKAGES_USED_IN_TESTS_USE_SUBDIRS_=true \
    _R_CHECK_SHLIB_OPENMP_FLAGS_=true \
    _R_CHECK_BROWSER_NONINTERACTIVE_=true \
    _R_CHECK_CODE_ASSIGN_TO_GLOBALENV_=true \
    _R_CHECK_CODE_DATA_INTO_GLOBALENV_=true \
    _R_CHECK_PKG_SIZES_=true \
    _R_CHECK_LIMIT_CORES_=true \
    _R_S3_METHOD_LOOKUP_BASEENV_AFTER_GLOBALENV_=true \
    _R_CHECK_AUTOCONF_=true \
    _R_CHECK_THINGS_IN_CHECK_DIR_=true \
    _R_CHECK_THINGS_IN_TEMP_DIR_=true \
    _R_CHECK_THINGS_IN_TEMP_DIR_EXCLUDE_="^ompi" \
    _R_CHECK_BASHISMS_=true \
    _R_CHECK_ORPHANED_=true \
    _R_CHECK_DEPENDS_ONLY_DATA_=true \
    _R_CHECK_XREFS_PKGS_ARE_DECLARED_=true \
    _R_CHECK_BOGUS_RETURN_=true \
    _R_CHECK_MATRIX_DATA_=TRUE \
    _R_CHECK_XREFS_USE_ALIASES_FROM_CRAN_=TRUE \
    _R_CHECK_MBCS_CONVERSION_FAILURE_=true \
    _R_CHECK_RD_VALIDATE_RD2HTML_=true \
    _R_CHECK_RD_MATH_RENDERING_=true \
    _R_CHECK_COMPILATION_FLAGS_KNOWN_="-Werror=implicit-function-declaration -Wno-error=enum-constexpr-conversion -Wp,-D_FORTIFY_SOURCE=3 -Wno-missing-template-arg-list-after-template-kw" \
    R_KNITR_OPTIONS='knitr.chunk.optipng="-quiet"' \
    OMP_THREAD_LIMIT=2

WORKDIR /work
CMD ["bash"]
