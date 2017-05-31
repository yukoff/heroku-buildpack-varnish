base_url=https://repo.varnish-cache.org/source
version=5.1.2
package=varnish-$(version).tar.gz
workdir=$(CACHE_DIR)/varnish-$(version)

.PHONY: default
default: install

$(CACHE_DIR):
	mkdir -p $(CACHE_DIR)

$(CACHE_DIR)/$(package): | $(CACHE_DIR)
	curl -o $@ "$(base_url)/$(package)"

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

$(workdir): $(CACHE_DIR)/$(package) | $(BUILD_DIR)
	tar -C $(dir $@) -zxf "$<"

# varnish ignores --without-rst2*, have to build ourselves and prove
# configure with the path
# following removed from configure:
# export PCRE_CFLAGS=-I$(BUILD_DIR)/include \
#        PCRE_LIBS="-lpcre -L$(BUILD_DIR)/lib"; \
.PHONY: configure
configure: | $(workdir) docutils
	cd $(workdir); \
    sh ./configure --prefix=/app/.heroku/varnish \
	               --with-rst2man=$(BUILD_DIR)/.heroku/varnish/bin/rst2man.py \
	               --with-rst2html=$(BUILD_DIR)/.heroku/varnish/bin/rst2html.py

.PHONY: compile
compile: | configure
	make -C $(workdir) PYTHONPATH=$(BUILD_DIR)/.heroku/varnish/lib/python2.7/site-packages

.PHONY: check
check: | compile
	make -C $(workdir) check

.PHONY: install
install: | compile
	cd $(workdir); \
	sh ./configure --prefix=/.heroku/varnish \
	               --with-rst2man=$(BUILD_DIR)/.heroku/varnish/bin/rst2man.py \
	               --with-rst2html=$(BUILD_DIR)/.heroku/varnish/bin/rst2html.py; \
	cd -; \
	make -C $(workdir) install DESTDIR=$(BUILD_DIR)

docutils:
	make -f $(WORK_DIR)/lib/Makefile.docutils install BUILD_DIR=$(BUILD_DIR)/.heroku/varnish CACHE_DIR=$(CACHE_DIR)

# libpcre:
# 	make -f $(WORK_DIR)/lib/Makefile.pcre install BUILD_DIR=$(BUILD_DIR) CACHE_DIR=$(CACHE_DIR)
