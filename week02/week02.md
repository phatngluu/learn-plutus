# Cannot start `cabal repl`
Error:
```
cabal: Could not resolve dependencies:
[__0] next goal: cardano-crypto-class (user goal)
[__0] rejecting: cardano-crypto-class-2.0.0 (conflict: pkg-config package
libsodium-any, not found in the pkg-config database)
[__0] fail (backjumping, conflict set: cardano-crypto-class)
```

Fix: https://github.com/input-output-hk/cardano-node/issues/1355
```
git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install

export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```

