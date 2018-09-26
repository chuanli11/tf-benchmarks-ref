CPU Benchmark
===

__Configuration__

* tf-nightly-gpu (20180814)
  - pip install tf-nightly-gpu==1.11.0.dev20180814
* Benchmark code: Based on TF's [benchmarks](https://github.com/tensorflow/benchmarks) (GitHub hash 8f95262).

Notice: 

* The default calculation of ```params.datasets_num_private_threads``` is problematic: when CPU has few cores ```datasets_num_private_threads``` can be set to 1 and this severely affects the speed. As a hacked-up solution we set ```num_monitoring_threads=0``` in benchmark_cnn.py, so ```datasets_num_private_threads = num_cpu_thread - 2 * num_gpus```.
* Alexnet (large parameter, low computation) causes significant bottleneck in real images training across all CPUs. Not be able to reproduce TF's reference Alexnet results on training real images. (```7900x``` is reasonablly close without distortions)
* Threadripper causes undeterministic segfault. For the successful runs it is on par with ```i7-7820X```
  * Solved by using higher speed RAM.
* Threadripeer CPU (1950X and 2990WX) use 128GB 2666 MHz RAM. 
* xeon CPUs has some issues with intensive pre-processing (extremely poor performance on Alexnet)
* ```E5-2686``` is tested with 4 V100 GPUs (AWS P3.8xlarge instance)
* all_reduce_spec is not fully tested (needs NCCL2). 

__Instructions__

Prepare Imagenet data. Here is a [link](https://drive.google.com/open?id=1JzF24uUa7D9fFeETrnNYMMMZ-9yNC0I5) to download a fraction of imagenet. Download and unzip it into ```~/data/imagenet_mini```

```
cd scripts/tf_cnn_benchmarks
./cpu_benchmark.sh
./cpu_report.sh
```

The results will be saved as ```summary.md```.

__Results__

| model | input size | param mem | feat. mem | flops | performance |
|-------|------------|--------------|----------------|-------|-------------|
| [resnet-50](reports/resnet-50.md) | 224 x 224 | 98 MB | 103 MB | 4 BFLOPs | 24.60 / 7.70 |
| [resnet-101](reports/resnet-101.md) | 224 x 224 | 170 MB | 155 MB | 8 BFLOPs | 23.40 / 7.00 |
| [resnet-152](reports/resnet-152.md) | 224 x 224 | 230 MB | 219 MB | 11 BFLOPs | 23.00 / 6.70 |
| [inception-v3](reports/inception-v3.md) | 299 x 299 | 91 MB | 89 MB | 6 BFLOPs | 22.55 / 6.44 |
| [vgg-vd-19](reports/vgg-vd-19.md) | 224 x 224 | 548 MB | 63 MB | 20 BFLOPs | 28.70 / 9.90 |
| [alexnet](reports/alexnet.md) | 227 x 227 | 233 MB | 3 MB | 1.5 BFLOPs | 41.80 / 19.20 |


**syn-parameter_server-4gpus**

CPU | i7-7820X | W-2133 | E5-1650 | i9-7900X | E5-2686 | i7-6850K | 1950X | 2990WX |
:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
resnet50 |725.24 |753.76 |782.59 | 766.19 |1250.24 |788.46 |710.47 |784.10 |
resnet101 |433.91 |446.11 |466.05 |444.68 |0 |474.53 |438.15 |457.60 |
resnet152 |268.94 |283.36 |291.24 |279.64 |0 |296.52 |274.58 |293.40 |
inception3 |458.14 |480.73 |484.92 |474.89 |0 |491.66 |493.03 |492.48 |
inception4 |177.05 |185.49 |193.79 |186.67 |0 |200.47 |185.11 |201.21 |
vgg16 |378.93 |416.86 |366.10 |416.89 |0 |428.64 |342.17 |372.75 |
alexnet |7703.72 |8507.48 |7431.79 |8483.71 |16517.47 |8566.71 |6655.69 |7488.62 |


**syn-replicated-4gpus**

CPU | i7-7820X | W-2133 | E5-1650 | i9-7900X | E5-2686 | i7-6850K | 1950X | 2990WX |
:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
resnet50 |736.60 |756.23 |770.03 |774.27 |1315.58 |788.76 |723.88 |772.01 |
resnet101 |448.89 |468.77 |465.87 |470.93 |0 |479.58 |457.67 |470.57 |
resnet152 |281.61 |301.15 |312.16 |306.62 |0 |314.56 |295.46 |314.17 |
inception3 |479.29 |495.29 |498.11 |498.95 |0 |507.20 |478.41 |502.56 |
inception4 |182.63 |200.40 |201.23 |205.33 |0 |209.27 |176.11 |205.91 |
vgg16 |365.29 |391.04 |369.65 |411.03 |0 |426.36 |341.26 |368.50 |
alexnet |7688.44 |8341.40 |7680.02 |8814.74 |15642.02 |8540.55 |6900.85 |7690.17 |


**real-parameter_server-distortions-4gpus**

CPU | i7-7820X | W-2133 | E5-1650 | i9-7900X | E5-2686 | i7-6850K | 1950X | 2990WX |
:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
resnet50 |710.10 |732.52 |767.77 |734.45 |1149.65 |769.18 |739.77 |712.26 |
resnet101 |428.46 |439.13 |458.97 |438.62 |0 |462.36 |446.51 |437.99 |
resnet152 |262.11 |271.09 |282.93 |269.80 |0 |283.31 |275.66 |284.10 |
inception3 |460.05 |470.72 |486.91 |470.89 |0 |494.61 |474.99 |451.94 |
inception4 |172.04 |185.68 |192.96 |183.72 |0 |191.03 |183.54 |195.24 |
vgg16 |369.07 |399.41 |355.33 |407.39 |0 |416.86 |349.06 |357.39 |
alexnet |2694.64 |1338.16 |1348.40 |3407.29 |1989.44 |1449.87 |5614.05 |4501.66 |


**real-parameter_server-4gpus**

CPU | i7-7820X | W-2133 | E5-1650 | i9-7900X | E5-2686 | i7-6850K | 1950X | 2990WX |
:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
resnet50 |713.49 |733.67 |764.16 |741.31 |1080.93 |772.52 |738.05 |724.36 |
resnet101 |427.18 |436.12 |456.76 |438.12 |0 |463.13 |446.46 |445.15 |
resnet152 |266.59 |274.20 |282.91 |279.41 |0 |284.07 |276.96 |282.93 |
inception3 |458.21 |468.41 |487.35 |473.27 |0 |494.94 |476.87 |460.85 |
inception4 |173.88 |182.96 |192.00 |190.41 |0 |191.12 |183.87 |196.51 |
vgg16 |371.47 |393.71 |359.44 |405.97 |0 |417.45 |351.29 |359.63 |
alexnet |4173.95 |2153.85 |1974.54 |5298.45 |2916.51 |2110.88 |5642.76 |5090.33 |


**real-replicated-distortions-4gpus**

CPU | i7-7820X | W-2133 | E5-1650 | i9-7900X | E5-2686 | i7-6850K | 1950X | 2990WX |
:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
resnet50 |720.86 |751.00 |761.44 |760.31 | |774.31 |728.28 |729.53 |
resnet101 |444.48 |463.43 |467.06 |466.99 |0 |475.72 |456.62 |460.22 |
resnet152 |279.23 |299.91 |307.23 |302.61 |0 |311.71 |293.59 |304.59 |
inception3 |478.32 |493.07 |497.98 |497.95 |0 |505.51 |474.58 |486.02 |
inception4 |180.22 |198.71 |201.28 |204.09 |0 |208.63 |174.81 |201.62 |
vgg16 |322.09 |354.82 |318.23 |369.76 |0 |385.04 |282.43 |314.64 |
alexnet |2669.96 |1336.53 |1344.23 |3375.17 | |1453.27 |5489.56 |4471.43 |


**real-replicated-4gpus**

CPU | i7-7820X | W-2133 | E5-1650 | i9-7900X | E5-2686 | i7-6850K | 1950X | 2990WX |
:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
resnet50 |722.68 |746.48 |757.99 |762.19 | |773.41 |729.39 |744.11 |
resnet101 |443.59 |463.48 |466.21 |468.62 |0 |476.42 |455.54 |455.44 |
resnet152 |278.53 |299.03 |307.24 |304.21 |0 |311.46 |293.81 |303.73 |
inception3 |475.84 |495.65 |494.23 |494.97 |0 |506.27 |475.33 |483.24 |
inception4 |180.90 |199.41 |200.07 |203.86 |0 |207.39 |175.00 |202.73 |
vgg16 |321.81 |353.39 |315.51 |367.78 |0 |385.97 |282.88 |317.07 |
alexnet |4111.26 |2150.01 |1972.83 |5239.91 | |2116.33 |5486.65 |4409.69 |

