# prjbureau / fuseconv.py

Alternative to ATMISP to generate SVF from JED

This is incomplete and not useful yet, just the upstream repo doesn't explain anything so I'm writing notes here.

```
git clone git@github.com:whitequark/prjbureau.git
cd prjbureau
sudo apt install yosys python3-sphinx python3-sphinx-rtd-theme python3-sphinx-argparse python3-bitarray
sh bootstrap.sh
```

This runs for a looooong time even on a i7-1280P with 64G ddr4  
Most of that seems to be generating html files under /doc

No idea if it works yet because the way it's written is not portable to use outside of the source tree.
