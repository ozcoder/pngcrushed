Glenn Randers-Pehrson (RIP) TODO
================================

(As noted below, some of the features that aren't yet implemented
   in _pngcrush_ are already available in **ImageMagick**; you can try a
   workflow that makes a first pass over the image with **ImageMagick**
   to select the bit depth, color type, interlacing, etc., and then makes
   another pass with _pngcrush_ to optimize the compression, and finally
   makes a pass with libpng's "**pngfix**" app to optimize the zlib CMF
   bytes.)

0. Make pngcrush optionally operate as a filter, writing the crushed PNG
   to standard output, if an output file is not given, and reading
   the source file from standard input if no input file is given.

1. Reset CINFO to reflect decoder's required window size (instead of
   libz-1.1.3 encoder's required window size, which is 262 bytes larger).
   See discussion about zlib in png-list archives for April 2001.
   libpng-1.2.9 does some of this and libpng-1.5.4 does better.
   But neither has access to the entire datastream, so pngcrush could
   do even better.

   This has no effect on the "crushed" filesize.  The reason for setting
   CINFO properly is to provide the *decoder* with information that will
   allow it to request only the minimum amount of memory required to decode
   the image (note that libpng-based decoders don't make use of this
   hint, because of the large number of files found in the wild that have
   incorrect CMF bytes).

   In the meantime, one can just run

   >      pc input.png crushed.png

   where the "pc" script is the following:

    ```
      #!/bin/sh
      cp $1 temp-$$.png
      for w in 512 1 2 4 8 16 32
      do
        pngcrush -ow -w $w -brute temp-$$.png
      done
      mv temp-$$.png $2
    ```

   It turns out that sometimes this finds a smaller compressed PNG
   than plain `pngcrush -brute input.png crushed.png` finds.

   There are several ways that pngcrush could implement this.

      a. Revise the bundled zlib to report the maximum window size that
      it actually used, then rewrite CINFO to contain the next power-of-two
      size equal or larger than the size.  This method would of course
      only work when pngcrush is built with the bundled zlib, and won't
      work with **zopfli** compression.

      b. Do additional trials after the best filter method, strategy,
      and compression level have been determined, using those settings
      and reducing the window size until the measured filesize increases,
      then choosing the smallest size which did not cause the filesize
      to increase.  This is expensive.

      c. After the trials are complete, replace CINFO with smaller
      settings, then attempt to decode the zlib datastream, and choose
      the smallest setting whose datastream can still be decoded
      successfully.  This is likely to be the simplest and fastest
      solution; however, it will only work with a version of libpng
      in which the decoder actually uses the CINFO hint.
      This seems to be the only method that would work with zopfli
      compression which always writes "7" (i.e., a 32k window) in CINFO.

      d. The simplest is to use **pngfix**, which comes with libpng16 and
      later, as a final step:

```
           pngcrush input.png temp.png
           pngfix --optimize --out=output.png temp.png
           # To do: find out if this works with zopfli output
```

2. Check for the possiblity of using the tRNS chunk instead of
      the full alpha channel.  If all of the transparent pixels are
      fully transparent, and they all have the same underlying color,
      and no opaque pixel has that same color, then write a tRNS
      chunk and reduce the color-type to 0 or 2. This is a lossless
      operation. **ImageMagick** already does this, as of version 6.7.0.
      If the lossy "-blacken" option is present, do that operation first.

3. Add choice of interlaced or non-interlaced output. Currently you
   can change interlaced to non-interlaced and vice versa by using
   **ImageMagick** before running pngcrush.  Note, when implementing this,
   disallow changing interlacing if APNG chunks are being copied.

4. Use a better compression algorithm for "deflating" (result must
   still be readable with zlib!)  e.g., http://en.wikipedia.org/wiki/7-Zip
   says that the 7-zip deflate compressor achieves better compression
   (smaller files) than zlib.  If tests show that this would be worth
   while, incorporate the 7-zip compressor as an optional alternative
   or additional method of pngcrush compression. See the GPL-licensed code
   at http://en.wikipedia.org/wiki/AdvanceCOMP and note that if this
   is incorporated in pngcrush, then pngcrush would have to be re-licensed,
   or released in two versions, one libpng-licensed and one GPL-licensed!

   Also consider Google's "zopfli" compressor, which is said to slow but
   achieves better compression.  It is Apache-2.0 licensed and available from
   a GIT repository at SourceForge (see https://code.google.com/p/zopfli/).
   See also the "pngzop" directory under the pmt.sourceforge.net project.
   Note paragraph 1.c above; zopfli always writes "7" in CINFO.  See
   my "pmt/pngzop" project at SourceForge and GitHub.

5. Optionally recognize any sRGB iCCP profile and replace it with the
   sRGB chunk.  Turn this option on if the "-reduce" option is on.  Also,
   if "-reduce" is on, delete any gAMA and cHRM chunks if the sRGB chunk
   is being written.

6. Accept "--long-option".  For starters, just accept -long_option
   and --long_option equally.

7. Implement a "copy_idat" option that simply copies the IDAT data,
   implemented as "-m 176". This will still repackage the IDAT chunks
   in a possibly different IDAT chunk size.

8. Implement palette-building (from ImageMagick-6.7.0 or later, minus
   the "PNG8" part) -- actually ImageMagick puts the transparent colors
   first, then the semitransparent colors, and finally the opaque colors,
   and does not sort colors by frequency of use but just adds them
   to the palette/colormap as it encounters them, so it might be improved.
   Also it might be made faster by using a hash table as was partially
   implemented in pngcrush-1.6.x.  If the latter is done, also port that
   back to ImageMagick/GraphicsMagick.  See also ppmhist from the NetPBM
   package which counts RGB pixels in an image; this and its supporting
   lib/libppmcmap.c would need to be revised to count RGBA pixels instead.

9. Improve the -help output and/or write a good man page.

10. Finish pplt (MNG partial palette) feature.

11. Remove text-handling and color-handling features and put
   those in a separate program or programs, to avoid unnecessary
   recompressing.  Note that in pngcrush-1.7.34, pngcrush began doing
   this extra work only once instead of for every trial, so the potential
   benefit in CPU savings is much smaller now.

12. Add a "pcRu" ancillary chunk that keeps track of the best method,
   methods already tried, and whether "loco crushing" was effective.

13. Try both transformed and untransformed colors when "-loco" is used.

14. Move the Photoshop-fixing stuff into a separate program.

15. GRR: More generally (superset of previous 3 items):  split into
   separate "edit" and "crush" programs (or functions).  Former is fully
   libpng-aware, much like current pngcrush; latter makes little or no use of
   libpng (maybe IDAT-compression parts only?), instead handling virtually
   all chunks as opaque binary blocks that are copied to output file _once_,
   with IDATs alone replaced (either by best in-memory result or by original
   _data_ resplit into bigger IDATs, if pngcrush cannot match/beat).  "edit"
   version should be similar to current code but more efficient:  make
   _one_ pass through args list, creating table of PNG_UINTs for removal;
   then make initial pass through PNG image, creating (in-order) table of
   all chunks (and byte offsets?) and marking each as "keep" or "remove"
   according to args table.  Could start with static table of 16 or 32 slots,
   then double size & copy if run out of room:  still O(n) algorithm.

16. With libpng17, png_write throws an "affirm()" with tc.format=256
   when attempting to write a sub-8-bit grayscale image.

17. Figure out why we aren't calling png_read_update_info() and fix.

18. Fix ADLER32 checksum handling in conjunction with iCCP chunk
   reading.

19. Fix Coverity "TOCTOU" warning about our "stat()" usage.

20. Warn about removing copyright and license info appearing in
   PNG text chunks.

21. Implement the "eXIf" chunk if it is approved by the PNG
   Development Group.  Optionally, remove preview and thumbnail
   images from the Exif profile contained in the eXIf chunk.
