# trtorhc

trtorchc is a compiler CLI application using the TRTorch compiler. It serves as an easy way to compile a
TorchScript Module with TRTorch from the command-line to quickly check support or as part of
a deployment pipeline. All basic features of the compiler are supported including post training
quantization (though you must already have a calibration cache file to use). The compiler can
output two formats, either a TorchScript program with the TensorRT engine embedded or
the TensorRT engine itself as a PLAN file.

All that is required to run the program after compilation is for C++ linking against libtrtorch.so
or in Python importing the trtorch package. All other aspects of using compiled modules are identical
to standard TorchScript. Load with `torch.jit.load()` and run like you would run any other module.


```
trtorchc [input_file_path] [output_file_path]
    [input_shapes...] {OPTIONS}

    TRTorch is a compiler for TorchScript, it will compile and optimize
    TorchScript programs to run on NVIDIA GPUs using TensorRT

  OPTIONS:

      -h, --help                        Display this help menu
      Verbiosity of the compiler
        -v, --verbose                     Dumps debugging information about the
                                          compilation process onto the console
        -w, --warnings                    Disables warnings generated during
                                          compilation onto the console (warnings
                                          are on by default)
        --info                            Dumps info messages generated during
                                          compilation onto the console
      --build-debuggable-engine         Creates a debuggable engine
      --use-strict-types                Restrict operating type to only use set
                                        default operation precision
                                        (op_precision)
      --allow-gpu-fallback              (Only used when targeting DLA
                                        (device-type)) Lets engine run layers on
                                        GPU if they are not supported on DLA
      -p[precision],
      --default-op-precision=[precision]
                                        Default operating precision for the
                                        engine (Int8 requires a
                                        calibration-cache argument) [ float |
                                        float32 | f32 | half | float16 | f16 |
                                        int8 | i8 ] (default: float)
      -d[type], --device-type=[type]    The type of device the engine should be
                                        built for [ gpu | dla ] (default: gpu)
      --engine-capability=[capability]  The type of device the engine should be
                                        built for [ default | safe_gpu |
                                        safe_dla ]
      --calibration-cache-file=[file_path]
                                        Path to calibration cache file to use
                                        for post training quantization
      --num-min-timing-iter=[num_iters] Number of minimization timing iterations
                                        used to select kernels
      --num-avg-timing-iters=[num_iters]
                                        Number of averaging timing iterations
                                        used to select kernels
      --workspace-size=[workspace_size] Maximum size of workspace given to
                                        TensorRT
      --max-batch-size=[max_batch_size] Maximum batch size (must be >= 1 to be
                                        set, 0 means not set)
      -t[threshold],
      --threshold=[threshold]           Maximum acceptable numerical deviation
                                        from standard torchscript output
                                        (default 2e-5)
      --save-engine                     Instead of compiling a full a
                                        TorchScript program, save the created
                                        engine to the path specified as the
                                        output path
      input_file_path                   Path to input TorchScript file
      output_file_path                  Path for compiled TorchScript (or
                                        TensorRT engine) file
      input_shapes...                   Sizes for inputs to engine, can either
                                        be a single size or a range defined by
                                        Min, Optimal, Max sizes, e.g.
                                        "(N,..,C,H,W)"
                                        "[(MIN_N,..,MIN_C,MIN_H,MIN_W);(OPT_N,..,OPT_C,OPT_H,OPT_W);(MAX_N,..,MAX_C,MAX_H,MAX_W)]"
      "--" can be used to terminate flag options and force all following
      arguments to be treated as positional options
```

e.g.
```
trtorchc tests/modules/ssd_traced.jit.pt ssd_trt.ts "[(1,3,300,300); (1,3,512,512); (1, 3, 1024, 1024)]" -p f16
```