
<!-- README.md is generated from README.Rmd. Please edit that file -->

# spng

<!-- badges: start -->

![](https://img.shields.io/badge/cool-useless-green.svg) [![R build
status](https://github.com/coolbutuseless/spng/workflows/R-CMD-check/badge.svg)](https://github.com/coolbutuseless/spng/actions)
<!-- badges: end -->

`spng` offers in-memory decompression of PNG images to vectors of raw
values.

This is useful if you have bytes representing a PNG image (e.g. from a
database) and need to decompress these to an array representation within
R.

`spng` is a R wrapper for
[libspng](https://github.com/randy408/libspng).

  - [libpng API docs](https://libspng.org/docs/api/)

## Installation

You can install from [GitHub](https://github.com/coolbutuseless/spng)
with:

``` r
# install.package('remotes')
remotes::install_github('coolbutuseless/spng')
```

## What’s in the box

  - `depng(raw_vec, fmt, flags)` - convert a vector of raw values
    containing a PNG image into a vector of raw bytes representing
    packed color values i.e. ABGR32 format

  - `get_info(raw_vec)` - interrogate a vector of raw values containing
    a PNG image to determine image information i.e. width, height,
    bit\_depth, color\_type, compression\_method, filter\_method,
    interlace\_method.

## Example: Decompress a PNG in memory

``` r
library(spng)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# A PNG file everyone should have!
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
png_file <- system.file("img", "Rlogo.png", package="png")
img <- magick::image_read(png_file)
img
```

<img src="man/figures/README-example-1.png" width="60%" />

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Read in the raw bytes
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
png_data <- readBin(png_file, 'raw', n = file.size(png_file))
png_data[1:100]
#>   [1] 89 50 4e 47 0d 0a 1a 0a 00 00 00 0d 49 48 44 52 00 00 00 64 00 00 00 4c 08
#>  [26] 06 00 00 00 9b 1d 12 0f 00 00 00 06 62 4b 47 44 00 ff 00 ff 00 ff a0 bd a7
#>  [51] 93 00 00 00 09 70 48 59 73 00 00 2e 23 00 00 2e 23 01 78 a5 3f 76 00 00 00
#>  [76] 07 74 49 4d 45 07 d5 02 10 10 08 0e 97 b9 27 bc 00 00 20 00 49 44 41 54 78
```

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Get info about the PNG 
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
(png_info <- spng::png_info(png_data))
#> $width
#> [1] 100
#> 
#> $height
#> [1] 76
#> 
#> $bit_depth
#> [1] 8
#> 
#> $color_type
#> [1] 6
#> 
#> $compression_method
#> [1] 0
#> 
#> $filter_method
#> [1] 0
#> 
#> $interlace_method
#> [1] 0
#> 
#> $color_desc
#> [1] "RGB + Alpha"
```

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Unpack the raw PNG bytes into RGBA 8-bits-per-color packed format. 
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
img_data <- spng::depng(png_data, fmt = spng_format$SPNG_FMT_RGBA8)
img_data[1:200]
#>   [1] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
#>  [26] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
#>  [51] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
#>  [76] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
#> [101] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
#> [126] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
#> [151] 00 00 00 00 00 00 00 00 00 00 c6 c8 c5 01 98 9b 96 13 8f 93 8c 31 87 8b 84
#> [176] 48 83 87 80 5d 81 85 7e 6e 80 84 7d 7c 7e 82 7a 84 81 85 7e 92 7e 83 7b 95
```

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Pick the green channel and plot as greyscale
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
N <- length(img_data)
mat <- matrix(img_data[seq(2, N, 4)], nrow = png_info$width, ncol = png_info$height)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Image had an alpha channel, let's replace that with white background
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
mat[mat == 0] <- as.raw(255)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# PNG bytes are in row-major order. R is in column major order
# so need to transpose to view correctly
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
plot(as.raster(t(mat)))
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="60%" />

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Construct an array by plucking the relevant bytes.
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
arr <- array(
  c(
    img_data[seq(1, N, 4)], # R
    img_data[seq(2, N, 4)], # G
    img_data[seq(3, N, 4)], # B
    img_data[seq(4, N, 4)]  # A
  ),
  dim = c(png_info$width, png_info$height, 4)
)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# PNG bytes are in row-major order. R is in column major order
# so need to transpose to view correctly
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
arr <- aperm(arr, c(2, 1, 3))
plot(as.raster(arr))
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="60%" />

# Use with `pixelweaver`

The vector of raw packed colour data returned by `{spng}` is in ABGR32
format.

`{pixelweaver}` v0.1.2 now supports the format and can directly ingest
the vector of raw values and convert to an array.

``` r
library(spng)
library(pixelweaver)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Read in raw bytes representing a PNG image
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
png_file <- system.file("img", "Rlogo.png", package="png")
png_data <- readBin(png_file, 'raw', n = file.size(png_file))
png_info <- spng::png_info(png_data)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Decode the PNG data to image data in-memory. Returned data is 
# packed color in ABGR32 format.
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
img_data <- spng::depng(png_data, fmt = spng_format$SPNG_FMT_RGBA8)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Convert the packed color directly into an R array with 4 channels (RGBA)
# This will also transpose the result into column-major representation
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
arr4 <- pixelweaver::packed_to_planar(
  packed_data = img_data, 
  format      = packed_fmt$ABGR32, # Packed color format
  nchannel    = 4,                 # Output array color depth
  width       = png_info$width, 
  height      = png_info$height
)


dim(arr4)
#> [1]  76 100   4
plot(as.raster(arr4))
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="60%" />

## Acknowledgements

  - R Core for developing and maintaining the language.
  - CRAN maintainers, for patiently shepherding packages onto CRAN and
    maintaining the repository
