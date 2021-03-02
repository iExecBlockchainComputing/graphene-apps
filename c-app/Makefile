# Use one of these commands to build the manifest for Bash:
#
# - make
# - make DEBUG=1
# - make SGX=1
# - make SGX=1 DEBUG=1
#
# Use `make clean` to remove Graphene-generated files.

# Relative path to Graphene root and key for enclave signing
GRAPHENEDIR ?= /home/oleg/graphene
SGX_SIGNER_KEY ?= $(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/enclave-key.pem

ifeq ($(DEBUG),1)
GRAPHENE_LOG_LEVEL = debug
else
GRAPHENE_LOG_LEVEL = error
endif

TARGET = app

RM = rm -f

ifneq ($(SGX_MODE), HW)
	URTS_LIB_NAME := sgx_urts_sim
	UAE_SERVICE_LIB := sgx_uae_service_sim
else
	URTS_LIB_NAME := sgx_urts
	UAE_SERVICE_LIB := sgx_uae_service
endif

INC:=-I$(SGX_SDK)/include -I../Include
LIB := -l$(URTS_LIB_NAME) -l$(UAE_SERVICE_LIB) -L$(SGX_SDK)/lib64 -lpthread
CXXFLAGS += $(INC) $(LIB)
CFLAGS += $(INC) $(LIB)

ifeq ($(SGX_DEBUG), 1)
	CXXFLAGS += -DDEBUG -UNDEBUG -UEDEBUG
	CFLAGS += -DDEBUG -UNDEBUG -UEDEBUG
else ifeq ($(SGX_PRERELEASE), 1)
	CXXFLAGS += -DEDEBUG -DNDEBUG -UDEBUG
	CFLAGS += -DEDEBUG -DNDEBUG -UDEBUG
else
	CXXFLAGS += -DNDEBUG -UEDEBUG -UDEBUG
	CFLAGS += -DNDEBUG -UEDEBUG -UDEBUG
endif

SRC_CPP=$(wildcard *.cpp)
SRC_C= $(wildcard ../util/*.c)

SRC_OBJ += $(SRC_CPP:.cpp=.o)
SRC_OBJ += $(SRC_C:.c=.o)


.PHONY = all clean


SGX_INC := -I$(SGX_SDK)/include

app.o: app.cpp
	@$(CXX) $(CXXFLAGS) -c $< -o $@
	@echo "CXX   <=  $<"

$(TARGET): $(SRC_OBJ)
	@$(CXX) $^ $(CXXFLAGS) -o $@
	#@mv $@ $(TOPDIR)/$(OUTDIR)/
	@echo "GEN  =>  $@"

.PHONY: all
all: $(TARGET) app.manifest | pal_loader
ifeq ($(SGX),1)
all: $(TARGET) app.manifest.sgx app.sig app.token
endif

include /home/oleg/graphene/Scripts/Makefile.configs

# Generate manifest rules for Bash dependencies.
# We'll duplicate some Glibc libraries (which Graphene provides in a customized version), but
# there's no harm in this.
PROGRAMS = app ls cat rm cp date
.INTERMEDIATE: trusted-libs
trusted-libs: /home/oleg/graphene/Examples/common_tools/get_deps.sh
	/home/oleg/graphene/Examples/common_tools/get_deps.sh $(foreach exec,$(PROGRAMS),$(shell which $(exec))) > $@

app.manifest: app.manifest.template trusted-libs
	(sed -e 's|$$(GRAPHENEDIR)|'"$(GRAPHENEDIR)"'|g' \
	     -e 's|$$(GRAPHENE_LOG_LEVEL)|'"$(GRAPHENE_LOG_LEVEL)"'|g' \
	     -e 's|$$(EXECDIR)|'"/home/oleg/graphene-apps/c-app"'|g' \
	     -e 's|$$(ARCH_LIBDIR)|'"$(ARCH_LIBDIR)"'|g' \
	     $<; \
	cat trusted-libs) > $@

# Generating the SGX-specific manifest (*.manifest.sgx), the enclave signature,
# and the token for enclave initialization.
app.manifest.sgx: app.manifest
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-sign \
		-libpal $(GRAPHENEDIR)/Runtime/libpal-Linux-SGX.so \
		-key $(SGX_SIGNER_KEY) \
		-manifest app.manifest -output $@

app.sig: app.manifest.sgx

app.token: app.sig
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-get-token \
		-output app.token -sig app.sig

pal_loader:
	ln -s $(GRAPHENEDIR)/Runtime/pal_loader $@

.PHONY: regression
regression: all
	@mkdir -p scripts/testdir

	./pal_loader ./app -c "ls" > OUTPUT
	@grep -q "Makefile" OUTPUT && echo "[ Success 1/6 ]"
	@rm OUTPUT

	./pal_loader ./app -c "cd scripts && app app.sh 1" > OUTPUT
	@grep -q "hello 1" OUTPUT      && echo "[ Success 2/6 ]"
	@grep -q "createdfile" OUTPUT  && echo "[ Success 3/6 ]"
	@grep -q "somefile" OUTPUT     && echo "[ Success 4/6 ]"
	@grep -q "current date" OUTPUT && echo "[ Success 5/6 ]"
	@rm OUTPUT

	./pal_loader ./app -c "cd scripts && bash bash_test.sh 3" > OUTPUT
	@grep -q "hello 3" OUTPUT      && echo "[ Success 6/6 ]"
	@rm OUTPUT

	@rm -rf scripts/testdir


.PHONY: clean
clean:
	$(RM) $(TARGET) $(TARGET).o *_u.c *_u.h *.manifest *.manifest.sgx *.token *.sig trusted-libs pal_loader OUTPUT scripts/testdir/*

.PHONY: distclean
distclean: clean
