#install fpm
#apt-get install ruby ruby-dev rubygems build-essential rpm
#gem install --no-document fpm
SHELL := /bin/bash

OP-NODE_VERSION := $(shell curl -s "https://api.github.com/repos/ethereum-optimism/optimism/releases/latest" | jq -r '.tag_name' | tr -d "v" | cut -d "/" -f 2)
BASE-NODE_VERSION := $(shell curl -s "https://api.github.com/repos/base-org/node/releases/latest" | jq -r '.tag_name' | tr -d "v" | cut -d "/" -f 2)

PKG_RELEASE := 0
PKG_NAME := optimism-op-node

PKG_DESCRIPTION := "Op-node Optimism client"
PKG_MAINTAINER := "Diego Losada <dlosada@ethereumonarm.com>"
CATEGORY := science
WEB_URL := https://github.com/ethereum-optimism/optimism/tree/develop/op-node

PKG_VENDOR := "Ethereum on ARM"
PKG_LICENSE := "GNU Lesser General Public License v3.0"

PKG_ARCH_DEB := riscv64
PKG_ARCH_RPM := riscv64

#PKG_AFTER_INSTALL := extras/postinstall

SOURCESDIR := sources/
OUTPUTDIR := ../../../packages

PKG_RPM := $(PKG_NAME)-$(OP-NODE_VERSION)-$(PKG_RELEASE)_$(PKG_ARCH_RPM).rpm
PKG_DEB := $(PKG_NAME)_$(OP-NODE_VERSION)-$(PKG_RELEASE)_$(PKG_ARCH_DEB).deb

FPM_DEB_OPTS := -s dir -n $(PKG_NAME) -v $(OP-NODE_VERSION) --license $(PKG_LICENSE)\
--vendor $(PKG_VENDOR) --iteration $(PKG_RELEASE) -C $(SOURCESDIR)\
--maintainer $(PKG_MAINTAINER) --description $(PKG_DESCRIPTION) -a $(PKG_ARCH_DEB)\
--category $(CATEGORY) --url $(WEB_URL) -p $(OUTPUTDIR) -x */.gitkeep\
--deb-systemd extras/op-node.service --deb-systemd extras/op-node-base.service\

FPM_RPM_OPTS := -s dir -n $(PKG_NAME) -v $(OP-NODE_VERSION) --license $(PKG_LICENSE)\
--vendor $(PKG_VENDOR) --iteration $(PKG_RELEASE) -C $(SOURCESDIR)\
--maintainer $(PKG_MAINTAINER) --description $(PKG_DESCRIPTION) -a $(PKG_ARCH_RPM)\
--category $(CATEGORY) --url $(WEB_URL) -p $(OUTPUTDIR) -x */.gitkeep\

all: prepare deb clean test

prepare: ## Download the latest binary and prepare the package files
	mkdir -p /tmp/op-node_tmp
	cd /tmp/op-node_tmp; git clone https://github.com/ethereum-optimism/optimism.git ; cd optimism/op-node ; env GO111MODULE=on GOOS=linux CC=riscv64-linux-gnu-gcc CXX=riscv64-linux-gnu-g++ GOARCH=riscv64 go build  -o ./build/bin/op-node ./cmd/main.go
	cp /tmp/op-node_tmp/optimism/op-node/build/bin/op-node sources/usr/bin/op-node
	# Get rollup.json and genesis file for Base
	#mkdir -p /tmp/op-node-base_tmp
	#cd /tmp/op-node_tmp; git clone https://github.com/base-org/node.git ; cd node ; git checkout v$(BASE-NODE_VERSION)
	#cp /tmp/op-node_tmp/node/mainnet/rollup.json sources/etc/ethereum 
	#cp /tmp/op-node_tmp/node/mainnet/genesis-l2.json sources/etc/ethereum

deb: ## Create deb package
	fpm -t deb -p $(PKG_DEB) $(FPM_DEB_OPTS)

rpm: ## Create rpm package
	fpm -t rpm -p $(PKG_RPM) $(FPM_RPM_OPTS)

.PHONY: clean test
clean: ## Clean up
	rm sources/usr/bin/op-node
	rm -rf /tmp/op-node_tmp
	rm -rf /tmp/op-node-base_tmp

test:
	@if [ -f $(OUTPUTDIR)/$(PKG_DEB) ]; then \
        printf "\033[32mPackage $(PKG_DEB) successfully created\033[0m\n"; \
    else \
        printf "\033[31mError: Package $(PKG_DEB) not found\033[0m\n"; \
        exit 1; \
    fi

help:  ## List targets.
	"$$(grep -hE '^\S+:.*##' $(MAKEFILE_LIST) | sed -e 's/:.*##\s*/:/' -e 's/^\(.\+\):\(.*\)/\\x1b[36m\1\\x1b[m:\2/' | column -c2 -t -s :)"
