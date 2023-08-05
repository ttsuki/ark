# 𝒂𝒓𝒌𝒂𝒏𝒂::ark

x86/x64, SSE/AVX Compiler intrinsic wrapper library for C++17

[MIT License](LICENSE) Copyright (c) 2020-2022 ttsuki

---

## Files

### `arkcpuid.h`

[`arkcpuid.h`](arkcpuid.h): CPU feature flag loader, provides `arkana::cpuid` namespace, defines `bool` constants representing the CPU's feature flags.

```cpp
#include <iostream>
#include <arkana-cpuid.h>

int main()
{
    if (arkna::cpuid::cpu_supports::AVX2)
        std::cout << "This CPU supports AVX2." << std::endl;
    else
        std::cout << "This CPU doesn't support AVX2." << std::endl;
}
```

```cpp
// cpuid.h

namespace arkana::cpuid::cpu_supports
{
	static const bool MMX = cpuflag(1, 3, 23);
	static const bool SSE = cpuflag(1, 3, 25);
	static const bool SSE2 = cpuflag(1, 3, 26);
	static const bool SSE3 = cpuflag(1, 2, 0);
	static const bool PCLMULQDQ = cpuflag(1, 2, 1);
	static const bool SSSE3 = cpuflag(1, 2, 9);
	static const bool SSE41 = cpuflag(1, 2, 19);
	static const bool SSE42 = cpuflag(1, 2, 20);
	static const bool AESNI = cpuflag(1, 2, 25);
	static const bool AVX = cpuflag(1, 2, 28);
	static const bool BMI = cpuflag(7, 1, 3);
	static const bool AVX2 = cpuflag(7, 1, 5);
	static const bool BMI2 = cpuflag(7, 1, 8);
}
```

---
### `arkxmm.h`

[`arkxmm.h`](arkxmm.h): **Strongly typed** SSE/AVX SIMD operation wrapper library, provides `arkxmm` namespace, defines 128,256 bit SIMD register types like `vi32x4`, `vu8x16`,..., (for 128 bit XMM register) and like `vu16x16`, `vu64x4` (for 256 bit AVX), and many operations for the types.

The operators provided by this library, calls naturally appropriate intrinsic function for each s/u sized type:

```cpp
int main()
{
    using namespace arkxmm;

    vi32x4 a = i32x4(1, 2, 3, 4); // a is arkxmm::vi32x4. (int 32bit x4)
    vi32x4 b = i32x4(5, 6, 7, 8); // b is arkxmm::vi32x4.
    vi32x4 c = (a + b) >> 1;      // `+` calls _mm_add_epi32, `>>` calls  _mm_srai_epi32 (because a and b are 32 bit type).

    vu32x4 d = reinterpret<vu32x4>(a); // reinterpret_cast: d is arkxmm::vu32x4. (unsigned 32bit x4)
    vu32x4 e = reinterpret<vu32x4>(b); // reinterpret_cast: d is arkxmm::vu32x4.
    vu32x4 f = (d + e) >> 1;           /// `+` calls _mm_add_epi32 as same, but `>>` calls _mm_srli_epi32 for unsigned.

    vu16x8 g = pack_sat_u(a, b); // pack vi32x4 x2 into vu16x8 (calls _mm_packus_epi32)
    vu16x8 h = unpack_hi(g, g);  // calls _mm_unpackhi_epi16 (because `g` is 16 bit type)
    vu16x8 i = (g + h) >> 1;     // `+` calls _mm_add_epi16, `>>` calls _mm_srli_epi16.

    auto x = a + d; // compile error! (no operator `+`` for vi32x4 and vu32x4)
}
```

Other usages of xmm.h in my code:
- [Convert NV12 surface to ARGB32](https://github.com/ttsuki/sandy/blob/develop/Sandy/MediaFoundation/SurfaceFormatConverter.cpp)
- [Change volume of 2ch stereo audio waveform](https://github.com/ttsuki/vse/blob/develop/vse/processing/WaveformProcessing.cpp)
- [High speed chacha20 cryptographic](https://github.com/ttsuki/chacha20poly1305/blob/develop/chacha20/chacha20.h)
- [High speed Camellia cryptographic with Intel AES-NI](https://github.com/ttsuki/arkana/blob/develop/arkana/camellia/camellia-avx2aesni.cpp)


```cpp
// xmm.h

namespace arkana::xmm {

// vector register types
struct vi8x16 { __m128i v; }; struct vu8x16 { __m128i v; };
struct vi16x8 { __m128i v; }; struct vu16x8 { __m128i v; };
struct vi32x4 { __m128i v; }; struct vu32x4 { __m128i v; };
struct vi64x2 { __m128i v; }; struct vu64x2 { __m128i v; };
struct vf32x4 { __m128 v; };  struct vf64x2 { __m128d v; };
struct vi8x32 { __m256i v; }; struct vu8x32 { __m256i v; };
struct vi16x16{ __m256i v; }; struct vu16x32{ __m256i v; };
struct vi32x8 { __m256i v; }; struct vu32x8 { __m256i v; };
struct vi64x4 { __m256i v; }; struct vu64x4 { __m256i v; };
struct vf32x8 { __m256 v; };  struct vf64x4 { __m256d v; };

template <class T> T zero(); // _zero
template <class T> T broadcast(T::element value); // _set1
template <class T> T from_values(T::element x0, x1,...); // _setr

// ctors
vi8x16 i8x16(int8_t x0, x1,..., x15) -> from_values<vi8x16>(x0...x15);
vu8x16 u8x16(uint8_t x0, x1,..., x15) -> from_values<vu8x16>(x0...x15);
vi16x8 i16x8(uint16_t x0, x1,..., x7) -> from_values<vi16x8>(x0...x7);
// ...
vf64x4 f64x4(double x0, x1,...,x3) -> ...

template <class T> load_u(const void* src); // _loadu
template <class T> store_u(void* dst, std::decay_t<T> v); // _storeu
template <class T> std::array<...> to_array(T v);

// for T in types
vi8x16 operator +(vi8x16 a, vi8x16 b);
vu8x16 operator +(vu8x16 a, vu8x16 b);
   ...
// and many operations are wrapped, see xmm.h directly.

}

```

---
### `arkintrinsic.h`

[`arkintrinsic.h`](arkintrinsic.h): Type conversion, Bit-manipulation, Multi-precision arithmetic helper, provides `arkana::intrinsics` namespace.

```cpp
// arkintrinsic.h

namespace arkana::intrinsic {

/// Loads T from unaligned memory pointer
template <class T> T load_u(const void *);
/// Stores T to unaligned memory pointer
template <class T> void store_u(void* d, const std::decay_t<T>& s);
/// bit cast
template <class To, class From> To bit_cast(const From& f);
/// type_punning_cast
template <class To, class From> std::remove_reference_t<To>& type_punning_cast(From& f);
/// secure_memzero
void secure_memzero(uint8_t* memory, size_t count);
template <class T> void secure_be_zero(T& t);

//// arithmetic ////

/// 128 bit unsigned integer
struct uint64x2_t { uint64_t l, h; };

using carry_flag_t = unsigned char;
carry_flag_t adc(carry_flag_t cf, uint_t& a, uint_t b); // adc for 32,64,128 bit
carry_flag_t sbb(carry_flag_t cf, uint_t& a, uint_t b); // sbb for 32,64,128 bit
auto muld(uint_t a, uint_t b); // muld for 32x32->64, 64x64->128 bit

//// bit-maniplation ////
uint_t rotl(uint_t v, int i); // for 8,16,32,64,128 bit
uint_t rotr(uint_t v, int i); // for 8,16,32,64,128 bit
uint_t byteswap(uint_t v, int i); // for 16,32,64,128 bit
uint_t shld(uint_t l, uint_t h, int i); // for 32,64 bit
uint_t shrd(uint_t l, uint_t h, int i); // for 32,64 bit
```

---
