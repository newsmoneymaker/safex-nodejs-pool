# Needs librandomx built from SafexCash's OWN node source (external/randomx submodule in
# https://github.com/safex/safexcore) -- never reuse another coin's prebuilt rxhash/librandomx, see
# critical-wrong-randomx-variant-bug memory. The only difference from stock RandomX is the Argon2 salt
# ("RandomSFX\x01" vs upstream's "RandomX\x03"), baked into that submodule's own configuration.h.
# RANDOMX=/path/to/prefix make        (prefix has include/randomx.h and lib/librandomx.a)
RANDOMX ?= /usr/local
CXX ?= g++
rxhash: rxhash.cpp
	$(CXX) -O2 -std=c++11 -pthread -I$(RANDOMX)/include -o $@ $< $(RANDOMX)/lib/librandomx.a -lcrypto
clean:
	rm -f rxhash
