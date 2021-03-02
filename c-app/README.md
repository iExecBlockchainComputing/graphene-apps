# Requirements

Intel SGX SDK and Graphene must be install following:
https://graphene.readthedocs.io/en/latest/building.html

# C app

Don't forget to setup SGX SDK environments
```
source $SGXSDK_DIR/environment
```

Build with
```
SGX=1 make
```

Build output
```
app.cpp: In function ‘int main(int, char**)’:
app.cpp:48:14: warning: unused variable ‘ret_status’ [-Wunused-variable]
     uint32_t ret_status;
              ^~~~~~~~~~
app.cpp:49:18: warning: unused variable ‘status’ [-Wunused-variable]
     sgx_status_t status;
                  ^~~~~~
CXX   <=  app.cpp
#@mv app //
GEN  =>  app
/home/oleg/graphene/Examples/common_tools/get_deps.sh  /bin/ls /bin/cat /bin/rm /bin/cp /bin/date > trusted-libs
(sed -e 's|$(GRAPHENEDIR)|'"/home/oleg/graphene"'|g' \
     -e 's|$(GRAPHENE_LOG_LEVEL)|'"error"'|g' \
     -e 's|$(EXECDIR)|'"/home/oleg/graphene-apps/c-app"'|g' \
     -e 's|$(ARCH_LIBDIR)|'"/lib/x86_64-linux-gnu"'|g' \
     app.manifest.template; \
cat trusted-libs) > app.manifest
ln -s /home/oleg/graphene/Runtime/pal_loader pal_loader
/home/oleg/graphene/Pal/src/host/Linux-SGX/signer/pal-sgx-sign \
	-libpal /home/oleg/graphene/Runtime/libpal-Linux-SGX.so \
	-key /home/oleg/graphene/Pal/src/host/Linux-SGX/signer/enclave-key.pem \
	-manifest app.manifest -output app.manifest.sgx
Attributes:
    size:        0x10000000
    thread_num:  4
    isv_prod_id: 0
    isv_svn:     0
    attr.flags:  0600000000000000
    attr.xfrm:   0300000000000000
    misc_select: 00000000
    date:        2021-03-02
Trusted files:
    3f073d46964c430d9f55fd7f1f90f9e77e1c8eee2c3fea6b8d6c6144637409e8 file:/home/oleg/graphene/Runtime/libsysdb.so
    8bcabdf6eab1923f8cee1ba48524e27be25b4f079c4e5ad55f40f5fc5fd70569 "/home/oleg/graphene-apps/c-app/app"
    9f29fb0f558d206603ef8a3306e0e70ee9da08e627b95338b3807566a07c7645 "file:/home/oleg/graphene/Runtime/ld-linux-x86-64.so.2"
    db12f74b8e21681ddedd91a7e9758d4e9fd723cc463d36870d3a0daf36f63a88 "file:/home/oleg/graphene/Runtime/libc.so.6"
    437e1b5a3d2f359be5d19f3f0c6b0bcea178ed38cd10befb1a0b2bc81dff554c "file:/home/oleg/graphene/Runtime/libm.so.6"
    a1d16ebae2c8a3d5d610f97a66d05755b30e087b5119c2c76ec7992ab4d38980 "file:/home/oleg/graphene/Runtime/libdl.so.2"
    aec455b366f7dffbea10fe8acf414c8847520f954ab9a899c7499d97d6fd52db "file:/home/oleg/graphene/Runtime/librt.so.1"
    f86255c9ac6390333f535ebd2b8d677860287054021be566e9ecab663cdeb3f8 "file:/home/oleg/graphene/Runtime/libutil.so.1"
    d2b54f770a6032d8aed8afc2d30b369eee2928092a407aeac00a6bf722727853 "file:/home/oleg/graphene/Runtime/libpthread.so.0"
    c78ab73d4800d19413945e938453a883fbc7b2010c874bbf5d6124ade060004b "file:/lib/x86_64-linux-gnu/libnss_compat.so.2"
    38d6029b89233409f935dcd32b0561e2ceda8e7fd1aa90d97aa836f6027e999d "file:/lib/x86_64-linux-gnu/libnss_files.so.2"
    33c429d3e8f272389c7d2ac8bf2b98c5eb338f6a24b898182498b7bf7c9fb939 "file:/lib/x86_64-linux-gnu/libnss_nis.so.2"
    fb0986a5026f20b337c07d17d4746cf8b9f3804848c44972efd4db09ac2f099e "file:/lib/x86_64-linux-gnu/libacl.so.1"
    da4544721850c1466cdc58f9540c28b26e2ec3dc6b0d2b37bf8f2ec74c6aa910 "file:/lib/x86_64-linux-gnu/libattr.so.1"
    467d8d5596e31cec78cdcde0c589bd04c031ec36598531bfd77e346ac447d9d6 "file:/lib/x86_64-linux-gnu/libc.so.6"
    398f33bacb604813ef22a5ae10b73a319180a6467a5e3e60a412dd271b2aff3b "file:/lib/x86_64-linux-gnu/libdl.so.2"
    d14920b8c1cb4e4cbe5f7c9ad23f6d9cf7b4ab1ddce1a29c9fc766b51c29ebea "file:/lib/x86_64-linux-gnu/libpcre.so.3"
    99c0999f13790d4365986180bc2e6dccbabd4d9b874a76799014c0c397e8f02a "file:/lib/x86_64-linux-gnu/libpthread.so.0"
    1307a5b52d35d96cee2b4fa1bb037b0495ee5c87f1f96727026a718ac4da8b0d "file:/lib/x86_64-linux-gnu/libselinux.so.1"
    56e0140608ed2623cf88d08d209e746b09ec48919af02f18976c0dfe594a4a57 "file:/lib64/ld-linux-x86-64.so.2"
Memory:
    000000000fffe000-0000000010000000 [REG:R--] (manifest) measured
    000000000ffde000-000000000fffe000 [REG:RW-] (ssa) measured
    000000000ffda000-000000000ffde000 [TCS:---] (tcs) measured
    000000000ffd6000-000000000ffda000 [REG:RW-] (tls) measured
    000000000ff96000-000000000ffd6000 [REG:RW-] (stack) measured
    000000000ff56000-000000000ff96000 [REG:RW-] (stack) measured
    000000000ff16000-000000000ff56000 [REG:RW-] (stack) measured
    000000000fed6000-000000000ff16000 [REG:RW-] (stack) measured
    000000000fec6000-000000000fed6000 [REG:RW-] (sig_stack) measured
    000000000feb6000-000000000fec6000 [REG:RW-] (sig_stack) measured
    000000000fea6000-000000000feb6000 [REG:RW-] (sig_stack) measured
    000000000fe96000-000000000fea6000 [REG:RW-] (sig_stack) measured
    000000000ba77000-000000000babe000 [REG:R-X] (code) measured
    000000000babe000-000000000fe96000 [REG:RW-] (data) measured
    0000000000010000-000000000ba77000 [REG:RWX] (free)
Measurement:
    6e9b53de2ef00fafae8b73c03b326942f5bb926fef8ea4c8180b78221d0b1ec9
/home/oleg/graphene/Pal/src/host/Linux-SGX/signer/pal-sgx-get-token \
	-output app.token -sig app.sig
lAttributes:
    mr_enclave:  6e9b53de2ef00fafae8b73c03b326942f5bb926fef8ea4c8180b78221d0b1ec9
    mr_signer:   cbcfe0bbc492e8b745f77353469f8f21cbea09130555c6f967241e273e08e037
    isv_prod_id: 0
    isv_svn:     0
    attr.flags:  0600000000000000
    attr.xfrm:   1f00000000000000
    misc_select: 00000000
    misc_mask:   00000000
    modulus:     bf361f4ae2badf6fc91402713e36bf98...
    exponent:    3
    signature:   893345d529c2cd4009d61aa9f7dbc0a4...
    date:        2021-03-02
rm trusted-libs

```

