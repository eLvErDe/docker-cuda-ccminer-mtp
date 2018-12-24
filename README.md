# nvidia-docker image for DJM34 CCMiner MTP fork (Zcoin new algo)

This image uses [ZCoin CCMiner] from my own [Debian/Ubuntu mining packages repository].
It requires a CUDA compatible docker implementation so you should probably go
for [nvidia-docker].
It has also been tested successfully on [Mesos] 1.5.1.

This MTP algo need much more memory than usual, so if it crashes you're problably missing RAM either on video card or host system.

## -api variants

It provides the PHP files connecting the TCP api and embed Apache2+PHP as well as a Python wrapper I made myself to start both Apache2 and CCMiner.

```
nvidia-docker run -dt --restart=unless-stopped -p 4068:4068 -p 4069:4069 --name cuda-xvg-ccminer acecile/cuda-ccminer-mtp-cuda9-api /root/multi-process-launcher/multi-process-launcher.py --cmd '/root/start-apache2.sh 4068 4069' --cmd '/usr/bin/ccminer -a mtp -o stratum+tcp://zcoin.mintpond.com:3000 -u aAQuL87ZehGSvXetp3JCmZWXmnWfkdkC9Q.pkg -p x --api-bind=0.0.0.0:4068'
```

You can now access status page at http://you.host:4069 and live hashrate graphs at http://you.host:4069/websocket.htm (yes, .htm it's not a typo).
It's currently using modified version of the existing examples but I hope to get them merged soon in tpruvot's repository.

## Build images

```
git clone https://github.com/eLvErDe/docker-cuda-ccminer-mtp
cd docker-cuda-ccminer-mtp
docker build -t cuda-ccminer-mtp .
docker build . -t cuda-ccminer-mtp-api -f Dockerfile.api
docker build . -t cuda-ccminer-mtp-cuda9 -f Dockerfile.cuda9
docker build . -t cuda-ccminer-mtp-cuda9-api -f Dockerfile.cuda9.api
```

## Publish it somewhere

```
docker tag cuda-ccminer-mtp docker.domain.com/mining/cuda-ccminer-mtp
docker push docker.domain.com/mining/cuda-ccminer-mtp

```

## Test it (using dockerhub published image)

```
nvidia-docker pull acecile/cuda-ccminer-mtp:latest
nvidia-docker run -it --rm acecile/cuda-ccminer-mtp /usr/bin/ccminer --help
```

An example command line to mine Zcoin using its new MTP algorithm:
```
nvidia-docker pull acecile/cuda-ccminer-mtp-cuda9:latest
nvidia-docker run -it --rm -p 4068:4068 --name cuda-xzc-ccminer acecile/cuda-ccminer-mtp-cuda9 /usr/bin/ccminer -a mtp -o stratum+tcp://zcoin.mintpond.com:3000 -u aAQuL87ZehGSvXetp3JCmZWXmnWfkdkC9Q.pkg -p x --api-bind=0.0.0.0:4068
```

Ouput will looks like:
```
*** ccminer 1.1.5-djm34 for nVidia GPUs by djm34 ***
    Built with the nVidia CUDA Toolkit 9.1

  Originally based on Christian Buchner and Christian H. project based on tpruvot 1.8.4 release
  Include algos from alexis78, djm34, sp, tsiv and klausT.
  *** News (07/06/2018): MTP algo for ZCoin 

BTC donation address: 1NENYmxwZGHsKFmyjTc5WferTn5VTFb7Ze (djm34)
ZCoin donation address: aChWVb8CpgajadpLmiwDZvZaKizQgHxfh5 (djm34)

[2018-12-24 15:23:51] Starting on stratum+tcp://zcoin.mintpond.com:3000
[2018-12-24 15:23:51] NVML GPU monitoring enabled.
[2018-12-24 15:23:51] 1 miner thread started, using 'mtp' algorithm.
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.3. Set the 'ServerName' directive globally to suppress this message
[2018-12-24 15:23:52] MESSAGE FROM SERVER: Optimizing difficulty may take several minutes.
[2018-12-24 15:23:52] Stratum difficulty set to 1 (0.00000)
[2018-12-24 15:23:52] GPU #0: Intensity set to 16, 65536 cuda threads
Step 1 : Compute F(I) and store its T blocks X[1], X[2], ..., X[T] in the memory 
Step 2 : Compute the root Φ of the Merkle hash tree 
end Step 2 : Compute the root Φ of the Merkle hash tree 
filling memory
memory filled 
[2018-12-24 15:24:08] GPU #0: MSI GTX 1080 Ti, 1059.84 kH/s
[2018-12-24 15:24:12] GPU #0: MSI GTX 1080 Ti, 1052.14 kH/s
[2018-12-24 15:24:16] GPU #0: MSI GTX 1080 Ti, 1059.82 kH/s
[2018-12-24 15:24:20] GPU #0: MSI GTX 1080 Ti, 1048.86 kH/s
[2018-12-24 15:24:24] GPU #0: MSI GTX 1080 Ti, 1071.23 kH/s
[2018-12-24 15:24:28] GPU #0: MSI GTX 1080 Ti, 1054.18 kH/s
```

## Background job running forever

```
nvidia-docker run -dt --restart=unless-stopped -p 4068:4068 --name cuda-xzc-ccminer acecile/cuda-ccminer-mtp-cuda9 /usr/bin/ccminer -a mtp -o stratum+tcp://zcoin.mintpond.com:3000 -u aAQuL87ZehGSvXetp3JCmZWXmnWfkdkC9Q.pkg -p x --api-bind=0.0.0.0:4068
```

You can check the output using `docker logs cuda-xzc-ccminer -f` 


## Use it with Mesos/Marathon

Edit `mesos_marathon.json` to replace miner parameter, change application path as well as docker image address (if you dont want to use public docker image provided).
Then simply run (adapt application name here too):

```
curl -X PUT -u marathon\_username:marathon\_password --header 'Content-Type: application/json' "http://marathon.domain.com:8080/v2/apps/mining/cuda-ccminer-mtp?force=true" -d@./mesos\_marathon.json
```

You can check CUDA usage on the mesos slave (executor host) by running `nvidia-smi` there:

```
Wed Dec  6 00:21:09 2017       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 375.82                 Driver Version: 375.82                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  On   | 0000:82:00.0     Off |                  N/A |
| 51%   67C    P2   197W / 200W |   1729MiB / 11172MiB |     98%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|    0     21474    C   /usr/bin/ccminer                              1729MiB |
+-----------------------------------------------------------------------------+
```

[ZCoin CCMiner]: https://github.com/zcoinofficial/ccminer
[Debian/Ubuntu mining packages repository]: https://packages.le-vert.net/mining/
[nvidia-docker]: https://github.com/NVIDIA/nvidia-docker
[Mesos]: http://mesos.apache.org/documentation/latest/gpu-support/
