# Delphes to LCIO tool and miniDST 

This repository provides an alternative way of using [`delphes2lcio`](https://github.com/iLCSoft/LCIO/tree/master/examples/cpp/delphes2lcio) via `docker`. You can easily install `docker` for various systems [`here`](https://docs.docker.com/get-docker/).


## Example 1: Generate LCIO output from stdhep file

Before we start: 

1. Please download the input file from [`here`](https://syncandshare.desy.de/index.php/s/63j6EDZH6e9Ec8w)
2. Create `data` folder and put this file there
3. Download docker image: `docker pull ilcsoft/delphes2lcio-v2`. This might take time. However, this is something we do only *once*

Now we are ready to launch a *container*:

```bash
cd ~/delphes2lcio-docker
docker run -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD/data:/home/ilc/data --rm -it --user $(id -u) ilcsoft/delphes2lcio-v1 bash
```

This works for **Ubuntu**. For **MAC OSX**, you should follow instructions [`here`] ( section enabling graphics) regarding XQuarz. 

You are inside the container. Be aware that `$PWD/data` has been mapped to `/home/ilc/data` **inside** the container. In addition, we need to do 

```bash
source init_env.sh 
export DATA=/home/ilc/data
```

We are ready to generate `slcio` output:

```bash
DelphesSTDHEP2LCIO $DELPHES_DIR/cards/delphes_card_ILD.tcl $DATA/output.slcio $DATA/E250-TDR_ws.Pe2e2h.Gwhizard-1_95.eR.pL.I106480.001.stdhep
```

This LCIO file can be analyzed like any other LCIO file with the usual tools,e.g.

```bash
dumpevent $DATA/output.slcio 1 | less
lcio_event_counter $DATA/output.slcio
```
### Root Macros
Getting started with the analysis is to use ROOT macros

```bash
cd /home/ilc/delphes2lcio/examples
root
root [0] .x ./fill_histos_lcio.C("/home/ilc/data/output.slcio")
```
this creates an output root file with the same base name and path, i.e.

```bash
root $DATA/output.root 
root [1] hetotpfo->Draw()
```
you should see reconstructed particle energy (i.e double Higgs peak)


## Example 2: Higgs recoil mass via mini-DST


