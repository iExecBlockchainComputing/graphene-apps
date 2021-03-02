# assumes this makefile lies in cwd
PWD := $(shell pwd)

GRAPHENE_DIR = /home/oleg/graphene
SGX_SIGNER_KEY ?= $(GRAPHENE_DIR)/Pal/src/host/Linux-SGX/signer/enclave-key.pem

MARIADB_DIR = $(PWD)/mariadb_dir
MARIADB_TARGZ=mariadb-10.5.9-linux-systemd-x86_64.tar.gz
MARIADB_URL ?= http://mariadb.mirrors.ovh.net/MariaDB/mariadb-10.5.9/bintar-linux-systemd-x86_64/$(MARIADB_TARGZ)
MARIADB_SHA256 ?= 0c671104803faf5f13a4e16093b459d421fcc224555af5c4e545417426030258
MARIADB_VER = 10.5.9

DATA_DIR = $(PWD)/data
RUN_DIR = $(PWD)/run_dir

UBUNTU_VER = $(shell lsb_release --short --id)$(shell lsb_release --short --release)

ifeq ($(UBUNTU_VER), Ubuntu20.04)
else ifeq ($(UBUNTU_VER), Ubuntu18.04)
else ifeq ($(UBUNTU_VER), Ubuntu16.04)
else
$(error This example requires Ubuntu 16.04, 18.04 or 20.04)
endif

ifeq ($(DEBUG),1)
GRAPHENE_LOG_LEVEL = debug
else
GRAPHENE_LOG_LEVEL = error
endif


.PHONY: all
all: $(MARIADB_DIR)/bin/mariadb mariadb.manifest | $(RUN_DIR)/pal_loader $(DATA_DIR)/db
ifeq ($(SGX),1)
all: mariadb.manifest.sgx mariadb.sig mariadb.token
endif

include /home/oleg/graphene/Scripts/Makefile.configs

$(MARIADB_DIR)/bin/mariadb:
	$(GRAPHENE_DIR)/Scripts/download --output $(MARIADB_TARGZ) \
		--sha256 $(MARIADB_SHA256) --url $(MARIADB_URL)
	mkdir -p $(MARIADB_DIR)
	tar -C $(MARIADB_DIR) --strip-components=1 -xf $(MARIADB_TARGZ)

$(RUN_DIR):
	mkdir -p $@

.INTERMEDIATE: trusted-libs
trusted-libs: /home/oleg/graphene/Examples/common_tools/get_deps.sh $(MARIADB_DIR)/bin/mariadb
	/home/oleg/graphene/Examples/common_tools/get_deps.sh $(MARIADB_DIR)/bin/mariadb > $@

mariadb.manifest: mariadb.manifest.template trusted-libs | $(RUN_DIR)
	(sed -e 's|$$(GRAPHENE_DIR)|'"$(GRAPHENE_DIR)"'|g' \
	     -e 's|$$(GRAPHENE_LOG_LEVEL)|'"$(GRAPHENE_LOG_LEVEL)"'|g' \
	     -e 's|$$(DATA_DIR)|'"$(DATA_DIR)"'|g' \
	     -e 's|$$(MARIADB_DIR)|'"$(MARIADB_DIR)"'|g' \
	     -e 's|$$(MARIADB_VER)|'"$(MARIADB_VER)"'|g' \
	     -e 's|$$(ARCH_LIBDIR)|'"$(ARCH_LIBDIR)"'|g' \
	     $<; \
    cat trusted-libs) > $@

# Make on Ubuntu <= 20.04 doesn't support "Rules with Grouped Targets" (`&:`),
# we need to hack around.
mariadb.sig mariadb.manifest.sgx: sgx_outputs

.INTERMEDIATE: sgx_outputs
sgx_outputs: $(MARIADB_DIR)/bin/mariadb mariadb.manifest \
        $(GRAPHENE_DIR)/Runtime/libpal-Linux-SGX.so | $(RUN_DIR)
	$(GRAPHENE_DIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-sign \
		-output mariadb.manifest.sgx \
		-libpal $(GRAPHENE_DIR)/Runtime/libpal-Linux-SGX.so \
		-key $(SGX_SIGNER_KEY) \
		-manifest mariadb.manifest

mariadb.token: mariadb.sig
	$(GRAPHENE_DIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-get-token -output $@ -sig $<

$(RUN_DIR)/pal_loader: | $(RUN_DIR)
	ln -s $(GRAPHENE_DIR)/Runtime/pal_loader $@

$(DATA_DIR)/db:
	mkdir -p $@

.PHONY: check
check: all
	cd $(RUN_DIR) && DATA_DIR=$(DATA_DIR) sh $(PWD)/test_all_scenes.sh

.PHONY: clean
clean:
	$(RM) -r $(RUN_DIR) $(DATA_DIR)/db mariadb.manifest mariadb.manifest.sgx mariadb.sig \
	         mariadb.token trusted-libs

.PHONY: distclean
distclean: clean
	$(RM) -r $(MARIADB_DIR) $(MARIADB_TARGZ)

