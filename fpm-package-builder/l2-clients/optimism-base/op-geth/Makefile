#install fpm
#apt-get install ruby ruby-dev rubygems build-essential rpm
#gem install --no-document fpm
SHELL := /bin/bash

#OP-GETH_VERSION := $(shell date +"%y%m%d")
OP-GETH_VERSION := $(shell curl -s "https://api.github.com/repos/ethereum-optimism/op-geth/releases/latest" | jq -r '.tag_name' | tr -d "v")
PKG_RELEASE := 0
PKG_NAME := optimism-op-geth

PKG_DESCRIPTION := "Op-geth Optimism client"
PKG_MAINTAINER := "Diego Losada <dlosada@ethereumonarm.com>"
CATEGORY := science
WEB_URL := https://github.com/ethereum-optimism/op-geth

PKG_VENDOR := "Ethereum on ARM"
PKG_LICENSE := "GNU Lesser General Public License v3.0"

PKG_ARCH_DEB := riscv64
PKG_ARCH_RPM := riscv64

SOURCESDIR := sources/
OUTPUTDIR := ../../../packages

PKG_RPM := $(PKG_NAME)-$(OP-GETH_VERSION)-$(PKG_RELEASE)_$(PKG_ARCH_RPM).rpm
PKG_DEB := $(PKG_NAME)_$(OP-GETH_VERSION)-$(PKG_RELEASE)_$(PKG_ARCH_DEB).deb

FPM_DEB_OPTS := -s dir -n $(PKG_NAME) -v $(OP-GETH_VERSION) --license $(PKG_LICENSE)\
--vendor $(PKG_VENDOR) --iteration $(PKG_RELEASE) -C $(SOURCESDIR)\
--maintainer $(PKG_MAINTAINER) --description $(PKG_DESCRIPTION) -a $(PKG_ARCH_DEB)\
--category $(CATEGORY) --url $(WEB_URL) -p $(OUTPUTDIR) -x */.gitkeep\
--deb-systemd extras/op-geth.service --deb-systemd extras/op-geth-base.service\
--depends zstd

FPM_RPM_OPTS := -s dir -n $(PKG_NAME) -v $(OP-GETH_VERSION) --license $(PKG_LICENSE)\
--vendor $(PKG_VENDOR) --iteration $(PKG_RELEASE) -C $(SOURCESDIR)\
--maintainer $(PKG_MAINTAINER) --description $(PKG_DESCRIPTION) -a $(PKG_ARCH_RPM)\
--category $(CATEGORY) --url $(WEB_URL) -p $(OUTPUTDIR) -x */.gitkeep\

all: prepare deb clean

prepare: ## Download the latest binary and prepare the package files
	mkdir -p /tmp/op-geth_tmp
	cd /tmp/op-geth_tmp/ ; git clone https://github.com/ethereum-optimism/op-geth.git ; cd op-geth ; git checkout v$(OP-GETH_VERSION);  env GO111MODULE=on CGO_ENABLED=1 GOOS=linux CC=riscv64-linux-gnu-gcc CXX=riscv64-linux-gnu-g++ GOARCH=riscv64 go build  -o ./build/bin/op-geth ./cmd/geth
	cp /tmp/op-geth_tmp/op-geth/build/bin/op-geth sources/usr/bin/op-geth

deb: ## Create deb package
	echo $(PKG_DEB) $(FPM_DEB_OPTS)
	fpm -t deb -p $(PKG_DEB) $(FPM_DEB_OPTS)

rpm: ## Create rpm package
	fpm -t rpm -p $(PKG_RPM) $(FPM_RPM_OPTS)

.PHONY: clean test
clean: ## Clean up
	rm sources/usr/bin/op-geth
	rm -rf /tmp/op-geth_tmp/

test:
	@if [ -f $(OUTPUTDIR)/$(PKG_DEB) ]; then \
        printf "\033[32mPackage $(PKG_DEB) successfully created\033[0m\n"; \
    else \
        printf "\033[31mError: Package $(PKG_DEB) not found\033[0m\n"; \
        exit 1; \
    fi
	
help:  ## List targets.
	"$$(grep -hE '^\S+:.*##' $(MAKEFILE_LIST) | sed -e 's/:.*##\s*/:/' -e 's/^\(.\+\):\(.*\)/\\x1b[36m\1\\x1b[m:\2/' | column -c2 -t -s :)"
