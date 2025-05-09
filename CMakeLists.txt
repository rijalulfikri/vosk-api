# Tentukan versi minimum CMake yang diperlukan
cmake_minimum_required(VERSION 3.13)

# Definisikan nama proyek dan bahasa yang digunakan (C++)
project(VoskLib CXX)

# --- Konfigurasi Path ---
# Atur path root Kaldi dan OpenBLAS. Ubah ini sesuai dengan lokasi instalasi Anda.
set(KALDI_ROOT "/Volumes/F/vosk/opt/kaldi")
set(OPENBLAS_ROOT "${KALDI_ROOT}/tools/OpenBLAS/install") # Asumsi OpenBLAS ada di dalam tools Kaldi

# --- Pengaturan Kompilasi ---

# Atur standar C++ yang diperlukan (C++17)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Definisikan source files untuk library
set(VOSK_SOURCES
    src/recognizer.cc
    src/language_model.cc
    src/model.cc
    src/spk_model.cc
    src/vosk_api.cc
    src/postprocessor.cc
)

# Tambahkan library target (libvosk), tipenya SHARED (.so)
add_library(vosk SHARED ${VOSK_SOURCES})

# Terapkan opsi kompilasi ke target 'vosk'
# -g: Informasi debug
# -O3: Optimasi level 3
# -Wno-deprecated-declarations: Matikan peringatan deprecated
# -fPIC: Position Independent Code (diperlukan untuk shared library)
target_compile_options(vosk PRIVATE
    -g
    -O3
    -Wno-deprecated-declarations
    -fPIC
)

# Tambahkan definisi preprocessor ke target 'vosk'
# -DFST_NO_DYNAMIC_LINKING
target_compile_definitions(vosk PRIVATE
    FST_NO_DYNAMIC_LINKING
)

# Tambahkan include directories ke target 'vosk'
target_include_directories(vosk PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    /Volumes/F/vosk/opt/vosk-api/src # Direktori saat ini (.)
    ${KALDI_ROOT}/src
    ${KALDI_ROOT}/tools/openfst/include
    ${OPENBLAS_ROOT}/include
)

# --- Pengaturan Linker ---

# Definisikan path ke static library Kaldi dan dependencies lainnya
set(KALDI_LIBS
    ${KALDI_ROOT}/build/src/online2/libkaldi-online2.a
    ${KALDI_ROOT}/build/src/decoder/libkaldi-decoder.a
    ${KALDI_ROOT}/build/src/ivector/libkaldi-ivector.a
    ${KALDI_ROOT}/build/src/gmm/libkaldi-gmm.a
    ${KALDI_ROOT}/build/src/tree/libkaldi-tree.a
    ${KALDI_ROOT}/build/src/feat/libkaldi-feat.a
    ${KALDI_ROOT}/build/src/lat/libkaldi-lat.a
    ${KALDI_ROOT}/build/src/lm/libkaldi-lm.a
    ${KALDI_ROOT}/build/src/rnnlm/libkaldi-rnnlm.a
    ${KALDI_ROOT}/build/src/hmm/libkaldi-hmm.a
    ${KALDI_ROOT}/build/src/nnet3/libkaldi-nnet3.a
    ${KALDI_ROOT}/build/src/transform/libkaldi-transform.a
    # Anda mungki/buildn perlu mengecek apakah kaldi-cudamatrix.a diperlukan jika tidak menggunakan CUDA
    # Jika Anda *tidak* mengompilasi Kaldi dengan dukungan CUDA, library ini mungkin tidak ada atau tidak diperlukan.
    # Jika Anda *membutuhkannya* dan Kaldi dikompilasi tanpa CUDA, ini bisa menyebabkan error.
    # Hapus baris berikut jika tidak ada atau tidak diperlukan:
    ${KALDI_ROOT}/build/src/cudamatrix/libkaldi-cudamatrix.a
    ${KALDI_ROOT}/build/src/matrix/libkaldi-matrix.a
    ${KALDI_ROOT}/build/src/fstext/libkaldi-fstext.a
    ${KALDI_ROOT}/build/src/util/libkaldi-util.a
    ${KALDI_ROOT}/build/src/base/libkaldi-base.a
    ${KALDI_ROOT}/tools/openfst/lib/libfst.a
    ${KALDI_ROOT}/tools/openfst/lib/libfstngram.a
    ${OPENBLAS_ROOT}/lib/libopenblas.a
    # Library berikut mungkin adalah dependensi dari OpenBLAS atau Kaldi math
    # Periksa apakah path dan nama file ini benar untuk instalasi Anda
    ${OPENBLAS_ROOT}/lib/liblapack.a # Mungkin sudah termasuk dalam libopenblas.a
    ${OPENBLAS_ROOT}/lib/libblas.a   # Mungkin sudah termasuk dalam libopenblas.a
    ${OPENBLAS_ROOT}/lib/libf2c.a    # Atau dependensi lain seperti libgfortran, tergantung bagaimana BLAS/LAPACK dikompilasi
)

# Link library-library tersebut ke target 'vosk'
target_link_libraries(vosk PRIVATE ${KALDI_LIBS})

# Terapkan opsi linker ke target 'vosk'
# -s: Strip symbols (menghapus informasi debug dari hasil akhir)
# -g: Sertakan informasi debug (mungkin redundan jika sudah di compile options, tapi sesuai command line)
# -fPIC: (Biasanya tidak perlu di linker flag jika sudah di compile options, tapi ditambahkan untuk konsistensi)
# Catatan: -s dan -g biasanya saling eksklusif dalam konteks build akhir (Release vs Debug).
#          Menggunakan keduanya secara bersamaan di command line asli mungkin tidak standar.
#          CMake biasanya menangani stripping secara terpisah (misalnya dengan 'strip' tool setelah build).
#          Namun, untuk mereplikasi command line, kita tambahkan flag ini.
target_link_options(vosk PRIVATE
    # -s #Opsional: Hapus simbol debug dari library akhir => membuat library tidak bisa di-debug
    -g
    #-fPIC # Umumnya tidak diperlukan di sini
)

# Opsional: Atur nama output secara eksplisit jika diperlukan (CMake biasanya menanganinya)
# set_target_properties(vosk PROPERTIES OUTPUT_NAME "vosk") # Tanpa "lib" dan ".so"

# Opsional: Atur direktori output library jika tidak ingin di default build directory
set_target_properties(vosk PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

message(STATUS "Configuring Vosk Main Executable (vosk_main)")

add_executable(vosk_main c/test_vosk.cc) # MyApp adalah nama target executable
target_link_libraries(vosk_main PRIVATE vosk)



# Opsional: Tambahkan pesan status
message(STATUS "Configuring Vosk Library Build")
message(STATUS "Kaldi Root: ${KALDI_ROOT}")
message(STATUS "OpenBLAS Root: ${OPENBLAS_ROOT}")