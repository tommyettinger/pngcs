<h1> PNGCS: a PNG library in C#</h1>

**PNGCS** is a library to read and write **PNG** images.
It's fully written in C#, it's provided as a _Pngcs.dll_ assembly in two flavors:


  * **.Net 4.5**: _pngcs45.dll_ single DLL. This alternative,  new since version 1.1.4, provides better speed, and does not require an extra dll.

  * **.Net 2.0** and above: requires third party _CsharpZipLib.dll_ assembly (included). Better compatibility and slightly better compression ratio.

This library is a C# port of the similar [PNGJ](http://code.google.com/p/pngj) library, in Java

It provides a simple API for progressive (sequential line-oriented) reading and writing. It's specially useful for huge images, which one does not want to  load fully in memory.

It supports  **all PNG spec color models and bitdepths**:

> True color: 8-16 bpp, with and without alpha channel (RGB/RGBA)

> Grayscale:  1-2-4-8-16 bpp, with and without alpha channel

> Indexed: palette with 1-2-4-8 bits

Since version 1.1.3 it also supports interlaced images (hence, it can read now all valid PNG files).

It has full **metadata** support: it reads and writes all the standard [ancillary Chunks](http://www.w3.org/TR/PNG/#11Ancillary-chunks), and it allows to register additional custom chunks.

It has been tested (read and write) against the [full standard PNG test suite](http://www.schaik.com/pngsuite/).

A quick example of use: the sample code below reads a PNG image file (true colour RGB/RGBA, 8 or 16 bpp), decreases the red channel value by 50% and writes it into another PNG file.

```

 public static void DecreaseRed(String origFilename, String destFilename) {
            PngReader pngr = FileHelper.CreatePngReader(origFilename); // or you can use the constructor
            PngWriter pngw = FileHelper.CreatePngWriter(destFilename, pngr.ImgInfo, true); // idem
            Console.WriteLine(pngr.ToString()); // just information
            int chunkBehav = ChunkCopyBehaviour.COPY_ALL_SAFE; // tell to copy all 'safe' chunks
            pngw.CopyChunksFirst(pngr, chunkBehav);          // copy some metadata from reader 
            int channels = pngr.ImgInfo.Channels;
            if (channels < 3)
                throw new Exception("This example works only with RGB/RGBA images");
            for (int row = 0; row < pngr.ImgInfo.Rows; row++) {
                ImageLine l1 = pngr.ReadRowInt(row); // format: RGBRGB... or RGBARGBA...
                for (int j = 0; j < pngr.ImgInfo.Cols; j++)
                    l1.Scanline[j * channels] /= 2; // decrease red channel
                pngw.WriteRow(l1, row);
            }
            pngw.CopyChunksLast(pngr, chunkBehav); // metadata after the image pixels? can happen
            pngw.End(); // dont forget this
            pngr.End();
        }
```


The next example generates an RGB8 (orange-ish gradient) image and writes it, progressively, to a Stream. Because only an image line is allocated, this would allow the creation of very large images with low memory usage.
This also shows some metadata handling.

```
 ImageInfo imi = new ImageInfo(cols, rows, 8, false); // 8 bits per channel, no alpha 
            // open image for writing 
            PngWriter png = FileHelper.CreatePngWriter(filename, imi, true);
            // add some optional metadata (chunks)
            png.GetMetadata().SetDpi(100.0);
            png.GetMetadata().SetTimeNow(0); // 0 seconds fron now = now
            png.GetMetadata().SetText(PngChunkTextVar.KEY_Title, "Just a text image");
            PngChunk chunk = png.GetMetadata().SetText("my key", "my text .. bla bla");
            chunk.Priority = true; // this chunk will be written as soon as possible
            ImageLine iline = new ImageLine(imi);
            for (int col = 0; col < imi.Cols; col++) { // this line will be written to all rows  
                int r = 255;
                int g = 127;
                int b = 255 * col / imi.Cols;
                ImageLineHelper.SetPixel(iline , col, r, g, b); // orange-ish gradient
            }
            for (int row = 0; row < png.ImgInfo.Rows; row++) {
                png.WriteRow(iline, row);
            }
            png.End();

```

The project sources includes some other [samples](http://code.google.com/p/pngcs/source/browse/#git%2FSamplesTests).

This project is quite in sync with its sibling (or parent) [PNGJ](http://code.google.com/p/pngj) (same functionality in Java), and it's very similar even at the code level. In case of doubts, you are advised to check the docs there.