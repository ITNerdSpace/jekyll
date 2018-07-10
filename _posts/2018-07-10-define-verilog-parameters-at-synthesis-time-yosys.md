---
layout: post
published: true
author: Alexandre Dumont
last_modified_at: '2018-07-10 15:44 +0200'
title: Define Verilog parameters at synthesis time (yosys)
---
I was looking for a way to pass parameters to Yosys (that would end up in the Verilog source code) at synthesis time.

Unfortunately, Yosys doesn't seem to allow any -D flag to pass parameter that would be later used in `define, like we would do in C/C++.

So I came with another way.

Imagine we have a design described in `top.v`. In that file, I have defined a couple of macros, that I'll use later in the code (here they represent files that will be loaded with `$readmemh()`):

`
`define PROGRAM "test/test01/program"
`define ROMFILE "test/test01/ram"
`

What I'm looking for here is a way to synthesize the design so that I can pass alternative arguments, that represent alternative values (that will override the one in the Verilog code).

Note: *Although the parameters I have here are filenames, it would be the same with any kind of parameters that we can think of.*

## The Approach: top_wrapper

The way I did it is like this:

First, I have wrapped both `define` macros in my top.v within an `ifndef/endif` block: if they already have a value, don't set the default value (or in other words, if they are not defined yet, set them to a default value).

```
`ifndef PROGRAM
`define PROGRAM "test/Year-32/program"
`endif

`ifndef ROMFILE
`define ROMFILE "test/Year-32/ram"
`endif
```

The next step was to create a wrapper for my `top.v` file. The wrapper would look like this:

```
`define PROGRAM "test/Year-01/program"
`define ROMFILE "test/Year-01/ram"
`include "top.v"
```

It's very simple: the wrapper sets the override value (will see later how we do that), and then includes the top.v file.

The thing is we need to generate this wrapper somehow. For that I chose to use `m4`.

For that, I created a new file, `top_wrapper.m4` with this content:

```
changequote([,])dnl
`define PROGRAM "_PROGRAM_"
`define ROMFILE "_ROMFILE_"
`include "top.v"
```

Basically, the `.m4` file is a template, and using the `m4` cli we'll be able to generate the top_wrapper.v with the value we passed on the command line.

Before assembling all that in out Makefile, there's one more problem we need to solve:

- If I don't specify any parameter on the command line, I want to be able to build the bitstream from top.v
- If I specify PROGRAM and/or ROMFILE parameters at build time, I want make to build from top_wrapper.v (which then requires top.v)

Without doing anything special in that 2nd case yosys will fail with:

```
ERROR: Re-definition of module `\top' at...
```

To avoid this, I enclose the whole top.v with an ifndef,define/endif block, so that it it only read once by yosys:

```
`ifndef __TOP_MODULE__
`define __TOP_MODULE__

[original content of top.v goes here]

`endif // __TOP_MODULE__
```

Why create a wrapper instead of converting top module into the template and generating the `top.v` at build time with `m4`? Well the main reasons were:
- `top.v` is kept practically unchanged, and is still a Verilog file (no m4 macros in it) so I can build the design from it directly if I want/need.
- as `top.v` is a pure Verilog file, I can benefit from synthax highlighting in my favorite code editor.

## Changes in Makefile

Now let's look at how we put that together in our Makefile.

I have a variable DEPS that lists all my source files (dependencies).

I have added this:

```
M4_OPTIONS=
AUXFILES=

ifdef PROGRAM
  M4_OPTIONS += -D_PROGRAM_=$(PROGRAM)
  DEPS := $(filter-out top_wrapper.v,$(DEPS)) top_wrapper.v
  AUXFILES += $(PROGRAM)
endif

ifdef ROMFILE
  M4_OPTIONS += -D_ROMFILE_=$(ROMFILE)
  DEPS := $(filter-out top_wrapper.v,$(DEPS)) top_wrapper.v
  AUXFILES += $(ROMFILE)
endif
```

If PROGRAM (respectively ROMFILE) is defined on the command line when calling `make`:
- we add it to M4_OPTIONS
- we add the top_wrapper.v to DEPS variables (only once)
- we add the parameter file to AUXFILES (so that if the content has changed, it will rebuild all)

I then add this next block ([taken from here](https://stackoverflow.com/questions/3236145/force-gnu-make-to-rebuild-objects-affected-by-compiler-definition)), so that we'll rebuild the top_wrapper.v (and all dependent targets if the parameter (filenames) have changed), or the content of those file have changed:

```
.PHONY: .force
build.config: $(AUXFILES) .force
	@echo '$(AUXFILES)' | cmp -s - $@ || echo '$(AUXFILES)' > $@
```

And, this will actually create the wrapper with m4:

```
top_wrapper.v: top_wrapper.m4 build.config
	m4 $(M4_OPTIONS) top_wrapper.m4 > top_wrapper.v
```

Because the wrapper is added to DEPS (source files), nothing else needs to be changed in the Makefile in respect to yosys. The same target that synthesize the design is letf unchanged:

```
$(MODULE).bin: $(MODULE).pcf $(MODULE).v $(DEPS) $(AUXFILES) build.config
	
	yosys -p "synth_ice40 -top $(MODULE) -blif $(MODULE).blif $(YOSYSOPT)" \
              -l $(MODULE).log -q $(MODULE).v $(DEPS)  
	
	arachne-pnr -d $(MEMORY) -p $(MODULE).pcf $(MODULE).blif -o $(MODULE).pnr
	
	icepack $(MODULE).pnr $(MODULE).bin
```

Notice how $(DEPS) and $(AUXFILES) are specified in the target dependencies (so it includes the top_wrapper.v and also it gets rebuild when any of them are updated, or it fails if any is missing).

## How to you pass the arguments to make

THe way to pass the value to the parameters at make time is by using the `NAME=value` format, right from the command line. For example:

```
$ make PROGRAM=test/test02/program0 ROMFILE=test/test02/ram0 bin
```

If we run twice the same make (with the same parameter values), it won't rebuild everything (if sources haven't changed).

## Alternatives

I have found some alternatives (by Yosys' author, Clifford).Actually, I'm not sure if it can be considered different alternative or is it actualy the same one). They didn't work for me in that particular case
, although they are worth keeping for future reference, as they could be useful in some other use cases.

- [Is there some way to pass parameters (or command line arguments) to a Yosys script?](https://stackoverflow.com/questions/44463230/parameters-to-script)
- [Setting parameter value inside verilog file](https://www.reddit.com/r/yosys/comments/2lezwp/setting_parameter_value_inside_verilog_file/)
- [Override verilog parameters](https://github.com/YosysHQ/yosys/issues/132)

Furthermore, in both case it would also have required me to create a yosys script template where I would then replace the variable with it's value. So I think at the end, I would also need m4 (or similar mechanism. Right now I feel happier with my approach :).