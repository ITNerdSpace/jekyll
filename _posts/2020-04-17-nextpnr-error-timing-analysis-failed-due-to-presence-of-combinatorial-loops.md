---
layout: post
published: true
author: Alexandre Dumont
last_modified_at: '2020-04-17 17:55 +0200'
title: 'nextpnr error: timing analysis failed due to presence of combinatorial loops'
---
## Timing analysis failed due to presence of combinatorial loops

When switching from ArchnePnR to NextPnR I started to get some weird errors when synthesizing my FPGA projects. The same project was synthesizing apparently fine in arachnePnR, but I guess NextPnR is more picky about some aspect. I guess it's better anyway, so the more warnings and errors we can get and fix, the more I'll learn and the better I hope the design would be.

One error I got was:

> ERROR: timing analysis failed due to presence of combinatorial loops, incomplete specification of timing ports, etc.

My reaction was to look at nextpnr output for some more indication about where the problem could be, but I was totally blind. So I headed to the #yosys channel on FreeNode, where are the awesome devs of Yosys, and some other experts, which usually are very helpful.

Their first hint was that Nextpnr maybe be complaining because of a combinational loop in its input, that is in the output of yosys, so that I should look at the yosys output for possible warnings.

Also they asked me if I had latches in your design? I wasn't aware of having any latch, but I got another very useful hint: look for 'Latch inferred' in yosys log! That was spot on:

Indeed, grepping yosys' output log I could find that a nasty latch has been infered:

```nosynthax
Latch inferred for signal `\top.\hex_digit' from process `\top.$proc$top.v:66$35':
$auto$proc_dlatch.cc:417:proc_dlatch$495
```

Ok that was really something I could work with now! This pointed me to `hex_digit` in `top.v`. And there I could finally see it. Notice how `hex_digit` doesn't have a default value in the always() block (while `char_shown` does):

```nosynthax
    always @(*) begin
      char_shown = 8'h00;
      if (activevideo2) begin

        if ( px_x2 >> (3+`Zoom) == 10 && px_y2 >> (3+`Zoom) ==  8  )
        begin
          hex_digit = counter[3:0];
          char_shown = digit_ascii_code;
        end

        else if ( px_x2 >> (3+`Zoom) ==  9 && px_y2 >> (3+`Zoom) ==  8  )
        begin
          hex_digit = counter[7:4];
          char_shown = digit_ascii_code;
        end

      end
    end
```

After fixing it, like below, (notice the `hex_digit = 4'b 0;` that gives a default value) I could finally pass the nextpnr step and synthesize fine!

```nosynthax
    always @(*) begin
      char_shown = 8'h00;
      hex_digit = 4'b 0;
      
      if (activevideo2) begin

        if ( px_x2 >> (3+`Zoom) == 10 && px_y2 >> (3+`Zoom) ==  8  )
        begin
          hex_digit = counter[3:0];
          char_shown = digit_ascii_code;
        end

        else if ( px_x2 >> (3+`Zoom) ==  9 && px_y2 >> (3+`Zoom) ==  8  )
        begin
          hex_digit = counter[7:4];
          char_shown = digit_ascii_code;
        end

      end
    end
```

Another finally tip that might come handy in this situation, in case you can't find the origin of the error, is that you can run nextpnr with `--ignore-loops` and it will simply ignore the error.