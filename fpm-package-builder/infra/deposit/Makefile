#install fpm
#apt-get install ruby ruby-dev rubygems build-essential rpm
#gem install --no-document fpm
SHELL := /bin/bash

DEPOSIT_VERSION := $(shell curl -s "https://api.github.com/repos/ethereum/staking-deposit-cli/releases/latest" | jq -r '.tag_name'| tr -d "v")
DEPOSIT_HASH := $(shell curl -s "https://api.github.com/repos/ethereum/staking-deposit-cli/tags" | jq -r '.[0].commit.sha' | cut -c1-7)

PKG_RELEASE := 0
PKG_NAME := staking-deposit-cli

PKG_DESCRIPTION := "deposit-cli is a tool for creating EIP-2335 format BLS12-381 keystores and a corresponding deposit_data*.json file for Ethereum Staking Launchpad."
PKG_MAINTAINER := "Diego Losada <dlosada@ethereumonarm.com>"

CATEGORY := science
WEB_URL :=  https://github.com/ethereum/eth2.0-deposit-cli/

PKG_VENDOR := "Ethereum on ARM"
PKG_LICENSE := "Creative Commons Zero v1.0 Universal"

PKG_ARCH_DEB := riscv64
PKG_ARCH_RPM := riscv64

SOURCESDIR := sources/
OUTPUTDIR := ../../packages

PKG_RPM := $(PKG_NAME)-$(DEPOSIT_VERSION)-$(PKG_RELEASE)_$(PKG_ARCH_RPM).rpm
PKG_DEB := $(PKG_NAME)_$(DEPOSIT_VERSION)-$(PKG_RELEASE)_$(PKG_ARCH_DEB).deb

FPM_DEB_OPTS := -s dir -n $(PKG_NAME) -v $(DEPOSIT_VERSION) --license $(PKG_LICENSE)\
	--vendor $(PKG_VENDOR) --iteration $(PKG_RELEASE) -C $(SOURCESDIR) --maintainer $(PKG_MAINTAINER)\
	--description $(PKG_DESCRIPTION) -a $(PKG_ARCH_DEB) --category $(CATEGORY) --url $(WEB_URL) -p $(OUTPUTDIR) -x */.gitkeep\
	--conflicts eth2.0-deposit-cli

FPM_RPM_OPTS := -s dir -n $(PKG_NAME) -v $(DEPOSIT_VERSION) --license $(PKG_LICENSE)\
	--vendor $(PKG_VENDOR) --iteration $(PKG_RELEASE) -C $(SOURCESDIR) --maintainer $(PKG_MAINTAINER)\
	--description $(PKG_DESCRIPTION) -a $(PKG_ARCH_RPM) --category $(CATEGORY) --url $(WEB_URL) -p $(OUTPUTDIR) -x */.gitkeep

BUILD_ARCH := $(shell uname -p)

all: prepare deb clean test

prepare: ## Donwload the latest binary and compile the package files
	wget https://github.com/ethereum/staking-deposit-cli/releases/download/v$(DEPOSIT_VERSION)/staking_deposit-cli-$(DEPOSIT_HASH)-linux-arm64.tar.gz -O - | tar -xz -C sources/usr/bin --wildcards --absolute-names --no-anchored --strip-components 2
deb: ## Create deb package
	fpm -t deb -p $(PKG_DEB) $(FPM_DEB_OPTS)

rpm: ## Create rpm package
	fpm -t rpm -p $(PKG_RPM) $(FPM_RPM_OPTS)

.PHONY: clean test
clean: ## Clean up
	rm sources/usr/bin/deposit

test:
	@if [ -f $(OUTPUTDIR)/$(PKG_DEB) ]; then \
        printf "\033[32mPackage $(PKG_DEB) successfully created\033[0m\n"; \
    else \
        printf "\033[31mError: Package $(PKG_DEB) not found\033[0m\n"; \
        exit 1; \
    fi

help:  ## List targets.
	@echo -e "$$(grep -hE '^\S+:.*##' $(MAKEFILE_LIST) | sed -e 's/:.*##\s*/:/' -e 's/^\(.\+\):\(.*\)/\\x1b[36m\1\\x1b[m:\2/' | column -c2 -t -s :)"
