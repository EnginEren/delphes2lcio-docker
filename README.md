# Delphes to LCIO tool and miniDST 

This repository provides an alternative way of using [`delphes2lcio`](https://github.com/iLCSoft/LCIO/tree/master/examples/cpp/delphes2lcio) via `docker`. You can easily install `docker` for various systems [`here`](https://docs.docker.com/get-docker/).


## Example 1: Generate LCIO output from stdhep file

Before we start: 

1. Please download the input file from [`here`](https://syncandshare.desy.de/index.php/s/63j6EDZH6e9Ec8w)
2. Download docker image: `docker pull ilcsoft/delphes2lcio-v3`. This might take time but this is something we do only *once*
3. Clone the repository: `git clone https://github.com/EnginEren/delphes2lcio-docker.git`
4. Go the our main folder (`cd ~/delphes2lcio-docker`) and create a data folder (`mkdir data`)
5. Put the input file you downloaded in step#1 to the `data` folder 

Now we are ready to launch a *container*:

### Linux
```bash
docker run -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v $PWD/data:/home/ilc/data -v $PWD/scripts:/home/ilc/scripts --rm -it --user $(id -u) ilcsoft/delphes2lcio-v3 bash
```
### Mac OSX 
You should follow instructions [`here`](https://hub.docker.com/r/rootproject/root) ( section enabling graphics) regarding XQuarz. 
After following instructions, quit X11 and start a new terminal with: 
```bash
docker run --rm -it -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$ip:0 -v $PWD/data:/home/ilc/data -v $PWD/scripts:/home/ilc/scripts  --user $(id -u) ilcsoft/delphes2lcio-v3 bash
```

You are inside the container. Be aware that `$PWD/data` and `$PWD/scripts` have been mapped to `/home/ilc/data` and `/home/ilc/scripts` **inside** the container. In addition, we need to do 

```bash
source init_env.sh 
export DATA=/home/ilc/data
```

We are ready to generate `slcio` output:

```bash
DelphesSTDHEP2LCIO $DELPHES_DIR/cards/delphes_card_ILD.tcl $DATA/E250-Pe2e2h.slcio $DATA/E250-TDR_ws.Pe2e2h.Gwhizard-1_95.eR.pL.I106480.001.stdhep
```

This LCIO file can be analyzed like any other LCIO file with the usual tools,e.g.

```bash
dumpevent $DATA/E250-Pe2e2h.slcio 1 | less
lcio_event_counter $DATA/E250-Pe2e2h.slcio
```
### Root Macros
Getting started with the analysis is to use ROOT macros

```bash
cd /home/ilc/delphes2lcio/examples
root
root [0] .x ./fill_histos_lcio.C("/home/ilc/data/E250-Pe2e2h.slcio")
```
this creates an output root file with the same base name and path, i.e.

```bash
### First quit from your previous root session 
## root [1] .q
root $DATA/output.root 
root [1] hetotpfo->Draw()
```
you should see reconstructed particle energy (i.e double Higgs peak)


## Example 2: Higgs Recoil mass via mini-DST

This example is about calculating the Higgs recoil mass on Higgsstrahlung events (e+e- ---> ZH) with Z->mumu. 

Please open a new shell in your local and download a mini-DST file [`here`](https://desycloud.desy.de/index.php/s/5LmrjGWqziQfMe7) via your favorite browser. Then put it into `data` folder. 

Assuming that you're still in your container,
```bash
cd ~/scripts
root
.x higgs_recoil.C("/home/ilc/data/<YOUR-MINI-DST-FILE-NAME>", "OUTPUTNAME")
```

Now, let us create the same plot, but this time with the ZH signal and e+e- --->mumujj background, dominated by e+e- ---> ZZ ---> mumujj, both normalised to given values for the integrated luminosity and the beam polarisations.

By default, ILC event samples are generated with 100% beam polarisation. For
all allowed sign combinations (usually just the two opposite-sign combinations, P(e-,e+) = (-1,+1) and (+1,-1)).
Distributions for realistic polaristaion values are then created by weighting the events - [./examples/higgs_recoil_with_bkg.C](./examples/higgs_recoil_with_bkg.C) shows you how this works.

It reads four input mini-DST files:

* [ee -> ZH -> mumuH, P(e-,e+) = (-1,+1)](https://desycloud.desy.de/index.php/s/5LmrjGWqziQfMe7)
* [ee -> ZH -> mumuH, P(e-,e+) = (+1,-1)](https://desycloud.desy.de/index.php/s/3ZqPcGPELggW4bP)
* [ee -> ZZ -> mumujj, P(e-,e+) = (-1,+1)](https://desycloud.desy.de/index.php/s/9gKznqtSGcBKBWY)
* [ee -> ZZ -> mumujj, P(e-,e+) = (+1,-1)](https://desycloud.desy.de/index.php/s/3i3tj3adfMPfPaC)

Once you downloaded the input files and the macro, you can type 
```bash
cd ~/scripts
root
.x higgs_recoil_with_bkg.C();
```
in your ROOT session to get the resulting plot.
In this case with quite a lot of background, since no cuts are applied (apart from two muons being present).
The macro optionally takes the following arguments with the following default values:

```bash
const char* DIRNAME = "./", double lumi_target=900., double epol_target=-0.8, double ppol_target=+0.3, TString outname = "recoil_plot"
```

where DIRNAME is the directory which hosts the input mini-DST files.

# Now it is your turn
Try to improve the signal-to-background ratio by applying a cut on the sum of the b-likeliness values of the two jets.
For this, read in the ```Refined2Jets``` collection, check that it is there and contains 2 jets, and then get the b-likeliness values (MVA output between 0 and 1).
You find an example of how to access jets and b-tag information in [./examples/jet_btag.C](./examples/jet_btag.C).
Take a look at this (of course you can also run it if you like!) and modify your ```higgs_recoil_with_bkg.C``` such that the recoil mass histograms are only filled if the sum of the two b-likeliness values > 1.
