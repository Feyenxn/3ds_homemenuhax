Currently this is just a tool for simulating the Home Menu code handling theme-data decompression.

When Home Menu is starting up, it can load theme-data from the home-menu theme SD extdata. The flaw can be triggered from here.

Home Menu allocates a 0x2a0000-byte heap buffer using the ctrsdk heap code: offset 0x0 size 0x150000 is for the output decompressed data, offset 0x150000 size 0x150000 is for the input compressed data. Immediately after this buffer is a heap freemem memchunkhdr, successfully overwriting it results a crash(when the data written there is junk) in the heap memchunk handling code with the linked-lists.

The decompression code only has an input-size parameter, no output size parameter. Hence, the output size is not restricted/checked at all. Since the decompressed data is located before the compressed data, the buf overflow results in the input compressed data being overwritten first. Eventually this overflow will result in the input data actually being used by the decompression function being overwritten, which will later result in an error before the function ever writes to the memchunk-hdr.

Triggering overwriting of the memchunk after the buffer is difficult, due to the input-compressed-data overwrite described above. Doing so would require two layers of compression somehow, basically.

